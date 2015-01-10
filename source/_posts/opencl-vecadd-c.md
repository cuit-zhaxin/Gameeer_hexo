title: OpenCL初探:向量相加（C语言版）
date: 2015-01-10 16:42:39
categories: 技术
tags: OpenCL
---
<p/>

下文记录了**向量相加**实例代码,另外推荐一些参考资料:
- OpenCL Reference Pages:[https://www.khronos.org/registry/cl/sdk/1.2/docs/man/xhtml/](https://www.khronos.org/registry/cl/sdk/1.2/docs/man/xhtml/)
- OpenCL API 1.2 Reference Card:[https://www.khronos.org/files/opencl-1-2-quick-reference-card.pdf](https://www.khronos.org/files/opencl-1-2-quick-reference-card.pdf)
- OpenCL 2.0 Reference:[https://www.khronos.org/files/opencl20-quick-reference-card.pdf](https://www.khronos.org/files/opencl20-quick-reference-card.pdf)

<!-- more -->
```
#include <stdio.h>
#include <stdlib.h>
#include <CL/cl.h>

const char* programSource =
	"__kernel					\n"
	"void vecadd(__global int *A,			\n"
	"	     __global int *B,			\n"
	"	     __global int *C)			\n"
	"{			 	      		\n"
	"						\n"
	"	int idx = get_global_id(0);		\n"
	"						\n"
	"	C[idx] = A[idx] + B[idx];		\n"
	"}						\n"
	;


int main(){

	//主机数据
	int *A = NULL;
	int *B = NULL;
	int *C = NULL;

	//数组里的元素个数
	const int elements = 2048;
	//计算数据宽度
	size_t datasize = sizeof(int)*elements;

	//为输入输出数据分配空间
	A = (int *)malloc(datasize);
	B = (int *)malloc(datasize);
	C = (int *)malloc(datasize);

	//初始化输入数据
	int i;
	for (i = 0;i<elements;i++)
	{
		A[i] = i;
		B[i] = i;
	}

	cl_int status;
	//平台数
	cl_uint numPlatforms = 0;
	status = clGetPlatformIDs(0,NULL,&numPlatforms);
	//为每一个平台分配足够空间
	cl_platform_id* platforms = NULL;
	platforms = (cl_platform_id*)malloc(numPlatforms*sizeof(cl_platform_id));
	//获取平台ID
	status = clGetPlatformIDs(numPlatforms,platforms,NULL);

	//检索设备数
	cl_uint numDevices = 0;
	status = clGetDeviceIDs(platforms[0],CL_DEVICE_TYPE_ALL,0,NULL,&numDevices);
	//为每一个设备分配足够空间
	cl_device_id* devices;
	devices = (cl_device_id*)malloc(numDevices*sizeof(cl_device_id));
	//获取设备ID
	status = clGetDeviceIDs(platforms[0],CL_DEVICE_TYPE_ALL,numDevices,devices,NULL);

	//新建上下文
	cl_context context;
	context = clCreateContext(NULL,numDevices,devices,NULL,NULL,&status);

	//创建命令队列
	cl_command_queue cmdQueue;
	cmdQueue = clCreateCommandQueue(context,devices[0],0,&status);

	//创建设备缓存
	cl_mem bufA;
	bufA = clCreateBuffer(context,CL_MEM_READ_ONLY,datasize,NULL,&status);
	cl_mem bufB;
	bufB = clCreateBuffer(context,CL_MEM_READ_ONLY,datasize,NULL,&status);
	cl_mem bufC;
	bufC = clCreateBuffer(context,CL_MEM_READ_ONLY,datasize,NULL,&status);

	//将数组中数据写入设备缓存
	status = clEnqueueWriteBuffer(cmdQueue,bufA,CL_FALSE,0,datasize,A,0,NULL,NULL);
	status = clEnqueueWriteBuffer(cmdQueue,bufB,CL_FALSE,0,datasize,B,0,NULL,NULL);

	//创建程序
	cl_program program = clCreateProgramWithSource(context,1,(const char**)&programSource,NULL,&status);
	//为设备编译程序
	status = clBuildProgram(program,numDevices,devices,NULL,NULL,NULL);

	cl_kernel kernel = clCreateKernel(program,"vecadd",&status);

	//设置内核参数
	status = clSetKernelArg(kernel,0,sizeof(cl_mem),&bufA);
	status = clSetKernelArg(kernel,1,sizeof(cl_mem),&bufB);
	status = clSetKernelArg(kernel,2,sizeof(cl_mem),&bufC);

	size_t globalWorkSize[1];
	globalWorkSize[0] = elements;
	
	//执行内核
	status = clEnqueueNDRangeKernel(cmdQueue,kernel,1,NULL,globalWorkSize,NULL,0,NULL,NULL);

	//从设备缓存读取数据
	clEnqueueReadBuffer(cmdQueue,bufC,CL_TRUE,0,datasize,C,0,NULL,NULL);

	int result = 1;
	for (i = 0;i<elements;i++)
	{
		if (C[i] != i+i)
		{
			result = 0;
			break;
		}
	}
	if (result)
	{
		printf("Output is correct\n");
	}else{
		printf("Output is incorrect\n");
	}

	//释放OpenCL资源
	clReleaseKernel(kernel);
	clReleaseProgram(program);
	clReleaseCommandQueue(cmdQueue);
	clReleaseMemObject(bufA);
	clReleaseMemObject(bufB);
	clReleaseMemObject(bufC);
	clReleaseContext(context);

	//释放主机资源
	free(A);
	free(B);
	free(C);
	free(platforms);
	free(devices);
	system("pause");
	return 0;
}
```