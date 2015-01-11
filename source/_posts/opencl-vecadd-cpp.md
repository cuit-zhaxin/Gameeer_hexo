title: OpenCL初探:向量相加(C++)
date: 2015-01-11 11:12:23
categories: 开发
tags: OpenCL
---
<p/>

遇到问题了，当设备类型为**CL_DEVICE_TYPE_GPU**时捕获错误如下：
![](/images/vecadd_cpp_err1.png)

将设备类型改为**CL_DEVICE_TYPE_CPU**或者**CL_DEVICE_TYPE_ALL**,不再报错。

下文是**使用C++封装API实现向量相加**的源码:
首先需得下载 cl.hpp 文件，下载地址:[https://www.khronos.org/registry/cl/](https://www.khronos.org/registry/cl/)
<!--more-->
```
#define __CL_ENABLE_EXCEPTIONS

#include <CL/cl.hpp>
#include <iostream>
#include <fstream>
#include <string>

int main(){

	const int N_ELEMENTS = 1024;
	int* A = new int[N_ELEMENTS];
	int* B = new int[N_ELEMENTS];
	int* C = new int[N_ELEMENTS];

	for (int i=0;i<N_ELEMENTS;i++)
	{
		A[i] = i;
		B[i] = i;
	}

	try{
		std::vector<cl::Platform> platforms;
		cl::Platform::get(&platforms);

		std::vector<cl::Device> devices;
		platforms[0].getDevices(CL_DEVICE_TYPE_GPU,&devices);

		cl::Context context(devices);
		cl::CommandQueue queue = cl::CommandQueue(context,devices[0]);

		cl::Buffer bufferA = cl::Buffer(context,CL_MEM_READ_ONLY,N_ELEMENTS*sizeof(int));
		cl::Buffer bufferB = cl::Buffer(context,CL_MEM_READ_ONLY,N_ELEMENTS*sizeof(int));
		cl::Buffer bufferC = cl::Buffer(context,CL_MEM_WRITE_ONLY,N_ELEMENTS*sizeof(int));

		queue.enqueueWriteBuffer(bufferA,CL_TRUE,0,sizeof(int),A);
		queue.enqueueWriteBuffer(bufferB,CL_TRUE,0,sizeof(int),B);

		std::ifstream sourceFile("vector_add_kernel.cl");
		std::string sourceCode(std::istreambuf_iterator<char>(sourceFile),(std::istreambuf_iterator<char>()));
		cl::Program::Sources source(1,std::make_pair(sourceCode.c_str(),sourceCode.length()+1));
		cl::Program program = cl::Program(context,source);
		program.build(devices);

		cl::Kernel vecadd_kernel(program,"vecadd");
		vecadd_kernel.setArg(0,bufferA);
		vecadd_kernel.setArg(1,bufferB);
		vecadd_kernel.setArg(2,bufferC);
		cl::NDRange global(N_ELEMENTS);
		cl::NDRange local(256);
		queue.enqueueNDRangeKernel(vecadd_kernel,cl::NullRange,global,local);

		queue.enqueueReadBuffer(bufferC,CL_TRUE,0,N_ELEMENTS*sizeof(int),C);

		bool result = true;
		for (int i=0;i<N_ELEMENTS;i++)
		{
			C[i] = B[i] + C[i];
			if (C[i]!=i+i)
			{
				result = false;
			}
		}
		if (result)
		{
			std::cout<<"Success!"<<std::endl;
		} 
		else
		{
			std::cout<<"Failed!"<<std::endl;
		}
	}catch(cl::Error error){
		std::cout<<error.what()<<"("<<error.err()<<")"<<std::endl;
	}
	system("pause");
	return 0;
}
```