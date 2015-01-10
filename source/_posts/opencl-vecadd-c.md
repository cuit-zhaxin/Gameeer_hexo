title: OpenCL��̽:������ӣ�C���԰棩
date: 2015-01-10 16:42:39
categories: ����
tags: OpenCL
---
<p/>

���ļ�¼��C���԰��������Ӵ���
<!-- more -->
```
#include <stdio.h>
#include <stdlib.h>
#include <CL/cl.h>

const char* programSource =
	"__kernel	\n"
	"void vecadd(__global int *A,	\n"
	"			 __global int *B,	\n"
	"			 __global int *C)	\n"
	"{	\n"
	"	\n"
	"	int idx = get_global_id(0);	\n"
	"	\n"
	"	C[idx] = A[idx] + B[idx];	\n"
	"}								\n"
	;


int main(){

	//��������
	int *A = NULL;
	int *B = NULL;
	int *C = NULL;

	//�������Ԫ�ظ���
	const int elements = 2048;
	//�������ݿ��
	size_t datasize = sizeof(int)*elements;

	//Ϊ����������ݷ���ռ�
	A = (int *)malloc(datasize);
	B = (int *)malloc(datasize);
	C = (int *)malloc(datasize);

	//��ʼ����������
	int i;
	for (i = 0;i<elements;i++)
	{
		A[i] = i;
		B[i] = i;
	}

	cl_int status;
	//ƽ̨��
	cl_uint numPlatforms = 0;
	status = clGetPlatformIDs(0,NULL,&numPlatforms);
	//Ϊÿһ��ƽ̨�����㹻�ռ�
	cl_platform_id* platforms = NULL;
	platforms = (cl_platform_id*)malloc(numPlatforms*sizeof(cl_platform_id));
	//��ȡƽ̨ID
	status = clGetPlatformIDs(numPlatforms,platforms,NULL);

	//�����豸��
	cl_uint numDevices = 0;
	status = clGetDeviceIDs(platforms[0],CL_DEVICE_TYPE_ALL,0,NULL,&numDevices);
	//Ϊÿһ���豸�����㹻�ռ�
	cl_device_id* devices;
	devices = (cl_device_id*)malloc(numDevices*sizeof(cl_device_id));
	//��ȡ�豸ID
	status = clGetDeviceIDs(platforms[0],CL_DEVICE_TYPE_ALL,numDevices,devices,NULL);

	//�½�������
	cl_context context;
	context = clCreateContext(NULL,numDevices,devices,NULL,NULL,&status);

	//�����������
	cl_command_queue cmdQueue;
	cmdQueue = clCreateCommandQueue(context,devices[0],0,&status);

	//�����豸����
	cl_mem bufA;
	bufA = clCreateBuffer(context,CL_MEM_READ_ONLY,datasize,NULL,&status);
	cl_mem bufB;
	bufB = clCreateBuffer(context,CL_MEM_READ_ONLY,datasize,NULL,&status);
	cl_mem bufC;
	bufC = clCreateBuffer(context,CL_MEM_READ_ONLY,datasize,NULL,&status);

	//������������д���豸����
	status = clEnqueueWriteBuffer(cmdQueue,bufA,CL_FALSE,0,datasize,A,0,NULL,NULL);
	status = clEnqueueWriteBuffer(cmdQueue,bufB,CL_FALSE,0,datasize,B,0,NULL,NULL);

	//��������
	cl_program program = clCreateProgramWithSource(context,1,(const char**)&programSource,NULL,&status);
	//Ϊ�豸�������
	status = clBuildProgram(program,numDevices,devices,NULL,NULL,NULL);

	cl_kernel kernel = clCreateKernel(program,"vecadd",&status);

	//�����ں˲���
	status = clSetKernelArg(kernel,0,sizeof(cl_mem),&bufA);
	status = clSetKernelArg(kernel,1,sizeof(cl_mem),&bufB);
	status = clSetKernelArg(kernel,2,sizeof(cl_mem),&bufC);

	size_t globalWorkSize[1];
	globalWorkSize[0] = elements;
	
	//ִ���ں�
	status = clEnqueueNDRangeKernel(cmdQueue,kernel,1,NULL,globalWorkSize,NULL,0,NULL,NULL);

	//���豸�����ȡ����
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

	//�ͷ�OpenCL��Դ
	clReleaseKernel(kernel);
	clReleaseProgram(program);
	clReleaseCommandQueue(cmdQueue);
	clReleaseMemObject(bufA);
	clReleaseMemObject(bufB);
	clReleaseMemObject(bufC);
	clReleaseContext(context);

	//�ͷ�������Դ
	free(A);
	free(B);
	free(C);
	free(platforms);
	free(devices);
	system("pause");
	return 0;
}
```