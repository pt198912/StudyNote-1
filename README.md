# StudyNote-1

  CMake使用简介及CMakeList.txt编写


	在新建工程的第一步选择Native C++，工程建好后会在app\src\main\cpp目录下生成CMakeList.txt。

	设置CMake需要的最小版本

	#设置CMake需要的最小版本
	cmake_minimum_required(VERSION 3.4.1)
	1
	2
	添加源文件让CMake编译成共享库

	add_library(
		ffmpeg-cmd
		SHARED
		ffmpeg/ffmpeg-cmd.cpp ffmpeg/ffmpeg.c ffmpeg/cmdutils.c ffmpeg/ffmpeg_filter.c ffmpeg/ffmpeg_hw.c ffmpeg/ffmpeg_opt.c
	)

	#指定头文件所在路径，相对于CMakeList.txt所在路径
	include_directories(ffmpeg/)



	ffmpeg-cmd-指定共享库名称，库文件名称会自动加上前缀lib变成libffmpeg-cmd.so，但是加载的时候仍然使用指定的名称：

	System.loadLibrary("ffmpeg-cmd");
	1
	SHARED-指定生成共享库
	ffmpeg/ffmpeg-cmd.cpp ffmpeg/ffmpeg.c...指定源码路径，多个源文件用空格隔开，注意是相对CMakeList.txt的路径

	添加预编译的库

	add_library(avcodec #指定引入模块的名称
		SHARED
		IMPORTED
		)

	SET_TARGET_PROPERTIES(
		avcodec #指定模块名称
		PROPERTIES IMPORTED_LOCATION
		${PROJECT_SOURCE_DIR}/ffmpeg/prebuilt/${ANDROID_ABI}/libavcodec.so
	)

	#指定模块头文件相对路径，如果已经指定过相同路径不需要重复指定
	include_directories(ffmpeg/)
	${PROJECT_SOURCE_DIR}表示CMakeList.txt所在路径
	如果有多个版本ABI的库文件，在编译的时候${ANDROID_ABI}会被替换成相应的abi名称。比如在gradle中做了如下配置

	ndk {
		    abiFilters 'armeabi-v7a', 'arm64-v8a'
	}


	CMake在编译的时候会分别到以下路径去找：
	…src/main/cpp/ffmpeg/prebuilt/armeabi-v7a/libavcodec.so
	…src/main/cpp/ffmpeg/prebuilt/arm64-v8a/libavcodec.so

	添加NDK API

	添加日志支持

	find_library(
			log-lib
		log)
	1
	2
	3
	链接

	为了在我们自己的库中能够调用其它库函数，需要设置target_link_libraries(）

	target_link_libraries( 
		ffmpeg-cmd #我们自己的库

		avcodec
		swscale
		swresample
		postproc
		avutil
		avformat
		avfilter
		${log-lib})


	举个栗子

	下面的示例演示了如何引入预编译的FFmpeg供自己调用：

	cmake_minimum_required(VERSION 3.4.1)

	#add libavcodec
	add_library(avcodec
		SHARED
		IMPORTED
		)

	SET_TARGET_PROPERTIES(
		avcodec
		PROPERTIES IMPORTED_LOCATION
		${PROJECT_SOURCE_DIR}/ffmpeg/prebuilt/${ANDROID_ABI}/libavcodec.so
	)

	#add libavfilter
	add_library(avfilter
		SHARED
		IMPORTED
		)

	SET_TARGET_PROPERTIES(
		avfilter
		PROPERTIES IMPORTED_LOCATION
		${PROJECT_SOURCE_DIR}/ffmpeg/prebuilt/${ANDROID_ABI}/libavfilter.so
	)


	#add libavformat
	add_library(avformat
		SHARED
		IMPORTED
		)

	SET_TARGET_PROPERTIES(
		avformat
		PROPERTIES IMPORTED_LOCATION
		${PROJECT_SOURCE_DIR}/ffmpeg/prebuilt/${ANDROID_ABI}/libavformat.so
	)


	#add libavutil
	add_library(avutil
		SHARED
		IMPORTED
		)

	SET_TARGET_PROPERTIES(
		avutil
		PROPERTIES IMPORTED_LOCATION
		${PROJECT_SOURCE_DIR}/ffmpeg/prebuilt/${ANDROID_ABI}/libavutil.so
	)


	#add libpostproc
	add_library(postproc
		SHARED
		IMPORTED
		)

	SET_TARGET_PROPERTIES(
		postproc
		PROPERTIES IMPORTED_LOCATION
		${PROJECT_SOURCE_DIR}/ffmpeg/prebuilt/${ANDROID_ABI}/libpostproc.so
	)

	#add libswresample
	add_library(swresample
		SHARED
		IMPORTED
		)

	SET_TARGET_PROPERTIES(
		swresample
		PROPERTIES IMPORTED_LOCATION
		${PROJECT_SOURCE_DIR}/ffmpeg/prebuilt/${ANDROID_ABI}/libswresample.so
	)

	#add libswscale
	add_library(swscale
		SHARED
		IMPORTED
		)

	SET_TARGET_PROPERTIES(
		swscale
		PROPERTIES IMPORTED_LOCATION
		${PROJECT_SOURCE_DIR}/ffmpeg/prebuilt/${ANDROID_ABI}/libswscale.so
	)

	include_directories(ffmpeg/)


	find_library(
		log-lib
		log)

	add_library(
		ffmpeg-cmd
		SHARED
		ffmpeg/ffmpeg-cmd.cpp ffmpeg/ffmpeg.c ffmpeg/cmdutils.c ffmpeg/ffmpeg_filter.c ffmpeg/ffmpeg_hw.c ffmpeg/ffmpeg_opt.c
	)
	target_link_libraries( # Specifies the target library.
		ffmpeg-cmd

		avcodec
		swscale
		swresample
		postproc
		avutil
		avformat
		avfilter
		${log-lib})

	目录结构：


	可能的报错

	1.SET_TARGET_PROPERTIES()中库文件路径指定错误：

	ninja: error: 'ffmpeg/prebuilt/arm64-v8a/libswscale.so', needed by 'D:/Android/Demo/AVDemo/app/build/intermediates/cmake/debug/obj/arm64-v8a/libffmpeg-cmd.so', missing and no known rule to make it
	1
	2.没有指定需要的头文件或路径错误:

	fatal error: 'libavcodec/mathops.h' file not found

2、为什么looper.loop不会阻塞主线程
	
	https://www.jianshu.com/p/72c44d567640
