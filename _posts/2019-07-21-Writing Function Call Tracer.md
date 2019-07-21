---
layout: post
title: ! "Debugging Functions를 이용해서 Function Call Tracer 만들기"
excerpt_separator: <!--more-->
tags:
  - Vertex
  - Debugging Function
  - Call Tracer

---

동적 분석 도구를 직접 만들고 싶다!!!!!! 어떻게 만들지 ??????
마이크로소프트에서는 디버거를 구현하기위한 Debugging Function들을 윈도우 API로 제공합니다!
이를 이용해 야매 Call Tracer를 만들어보도록 하겠습니다.

<!--more-->

# 동적 분석과 정적 분석

실행 파일을 리버스 엔지니어링 할때 두 가지의 관점에서 진행할 수 있습니다.

바로 정적 분석과 동적 분석입니다.

정적 분석은 파일을 실행하지 않고 실행 파일의 메타데이터나 내부에 있는 어셈블리 코드만을 보고서 분석을 진행합니다.

동적 분석은 파일을 실행하면서 분석을 진행합니다. 실제 어셈블리 코드에서 어떤 파라메터를 통해 프로그램이 실행되는지 확인할 수 있습니다. 동적 분석 도구를 활용하면 어셈블리 코드를 보지 않더라도 어떤 윈도우 API가 실행되는지, 디스크 I/O, 네트워크 I/O를 자세하고 빠르게 프로그램의 행위를 확인할 수 있습니다.

|      |                          정적 분석                           |                          동적 분석                           |
| :--: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 장점 | 예상치 못한 실행파일의 행위로 인한 시스템의 피해를 미연에 방지 가능 | 짧은 시간에 프로세스에 대한 행위 정보를 많이 수집할 수 있음. 어셈블리 몰라도 분석이 어느정도 가능. |
| 단점 |                          어!렵!다!                           | 실행했는데 컴퓨터가 고장이 나면...? 인터넷에 접속해서 뭔가 기록을 남겨버리면?? |



# 동적 분석 도구를 만들기 위해서는?

프로그램을 실행하는 과정을 기록으로 남기기 위해서는 실행 흐름을 제어할 필요가 있습니다.

먼저 생각해볼 수 있는 방법은 API 후킹이 있습니다.  하지만 API 후킹을 통해 동적 분석을 하려고 하면....

관련 윈도우 API만 해도 상당히 많을것이고... 후킹을 자동화해도 소스코드가 방대해질 것 같았습니다.



하지만 디버거를 만든다면 어떨까요???

직접 간단한 디버거를 만들어서 모든 어셈블리 코드를 한줄한줄 확인할 수 있다면 더 구현이 간단할 것 같다는 생각을 했습니다.



# Debugging API로 간단 디버거 제작

```c++
ZeroMemory(&si, sizeof(si));
si.cb = sizeof(si);
ZeroMemory(&pi, sizeof(pi));
bool a;
a=CreateProcess(imageName.c_str(), NULL, NULL, NULL, FALSE,
	DEBUG_PROCESS, NULL, NULL, &si, &pi);
DebugBreakProcess(pi.hProcess);
```



CreateProcess에 DEBUG_PROCESS 플래그를 넣어서 실행해주시면 만들어진 프로그램에 대한 디버깅 권한을 자동으로 획득합니다.



```c++
	while (true)
	{
		if (!WaitForDebugEvent(&debug_event, INFINITE))
			return 0;
    	switch(debug_event.dwDebugEventCode){

        }
        ContinueDebugEvent(debug_event.dwProcessId,debug_event.dwThreadId, 
                           DBG_CONTINUE);
    }
```

디버깅 권한을 얻으면 자동으로 윈도우 커널이 Debugger(디버깅 하는 친구)와 Debuggee(디버깅 당하는 친구)를 짝지어줍니다. 그리고 디버기에서 뭔가 인터럽트가 발생하면 디버거가 이를 처리할 수 있게 도와줍니다.

![](https://i.imgur.com/2gucQ46.png)

dwDebugEventCode는 총 9개가 있구요. 이 이벤트 코드에 따라 debug_event를 처리하시면 됩니다.



```c++
		case CREATE_THREAD_DEBUG_EVENT:

			printf("Thread 0x%x (Id: %d) created at: 0x%x\n",
				(unsigned int)(debug_event.u.CreateThread.hThread),
				debug_event.dwThreadId,
				(unsigned int)debug_event.u.CreateThread.lpStartAddress);
			// Thread 0xc (Id: 7920) created at: 0x77b15e58
			//success=SetBreakPoint(pi.hProcess, (DWORD)debug_event.u.CreateThread.lpStartAddress);   //SOFTWARE BreakPoint 설정
			setTrapFlag(debug_event.u.CreateThread.hThread);
			ThreadHandles[debug_event.dwThreadId] = debug_event.u.CreateThread.hThread;     //ThreadId에 대한 핸들 저장
			ThreadCallDepth[debug_event.dwThreadId]=0;
			break;
```



저 8번 라인에 SetBreakPoint 함수는 사용자 정의 함수인데, 쓰레드의 시작 지점을 0xCC (int 3 어셈명령어)로 교체해주는 함수입니다.

원래 디버거를 직접 제작할때 0xCC로 해당 1바이트를 수정해서 Software BreakPoint를 건다고 하는데, 해당 브레이크포인트 지점이 멀티쓰레드를 마구 생성하고 소멸하는 지점에 있으면 이 Debugging Event Loop로는 처리가 불가능하더라구요..



```c++
bool SetBreakPoint(HANDLE hProcess ,DWORD addr) {
	DWORD dwStartAddress = addr;
	BYTE cInstruction,m_OriginalInstruction;
	DWORD dwReadBytes;
	bool success;

	// Read the first instruction
	ReadProcessMemory(hProcess, (void*)dwStartAddress, &cInstruction, 1, &dwReadBytes);

	// Save it!
	m_OriginalInstruction = cInstruction;

	// Replace it with Breakpoint
	cInstruction = 0xCC;
	WriteProcessMemory(hProcess, (void*)dwStartAddress, &cInstruction, 1, &dwReadBytes);
	success = FlushInstructionCache(hProcess, (void*)dwStartAddress, 1);
	softBreakPoint[addr] = m_OriginalInstruction;
	return success;
}
```

해당 어드레스에 있는 1바이트를 Unordered_map 에 저장한 다음에 거기를 0xCC로 덮어씌우는 코드입니다.



그럼 이 0xCC를 어떻게 브레이크포인트로 처리하냐?





EXCEPTION_DEBUG_EVENT코드에서

ExceptionCode가 EXCEPTION_BREAKPOINT면, 이 0xCC가 발동되서 나온거라고 보시면 됩니다.

근데 이 EXCEPTION_BREAKPOINT 이벤트는 저희가 SW 브레이크포인트를 건 지점만 딱 멈춰서 브포가 걸리는게 아닙니다. 가끔 그냥 쓰레드가 진행하다가 int 3을 만나서 이벤트가 뜨기도 합니다(-_-)

그래서 내가 브레이크포인트를 건 위치인지 확인하고 처리를 하시면 됩니다.

```c++
case EXCEPTION_DEBUG_EVENT:
	EXCEPTION_DEBUG_INFO& exception = debug_event.u.Exception;
	DWORD ExceptionPos = (DWORD)debug_event.u.Exception.ExceptionRecord.ExceptionAddress;

	switch (exception.ExceptionRecord.ExceptionCode)
	{
		case EXCEPTION_BREAKPOINT:
			if (softBreakPoint.count(ExceptionPos)) { //브레이크포인트 걸려있으면?
				success = RemoveBreakPoint(pi.hProcess,ExceptionPos);
				BackEIP(ThreadHandles[debug_event.dwThreadId]);
				setTrapFlag(ThreadHandles[debug_event.dwThreadId]);
			}
```

RemoveBreakPoint도 그냥 별거 없이 해당 위치에 0xCC대신 원래 바이트코드로 덮어씌워주는겁니다.

그리고 BackEIP는 이렇게 생겼습니다.

```c++
void BackEIP(HANDLE hThread) {
	CONTEXT lcContext;
	lcContext.ContextFlags = CONTEXT_ALL;
	GetThreadContext(hThread, &lcContext);
	lcContext.Eip--;
	SetThreadContext(hThread, &lcContext);
}
```

ThreadContext를 통해 쓰레드별로 레지스터를 제어할 수 있습니다.

EIP를 -1 해주는 함수입니다. 0xCC 를 원래대로 복구시키고 그대로 프로그램 실행하면 원래 있던 어셈블리 코드에 맨 앞 1바이트가 유실된 채로 실행되려고 하니.... 좋지 않은 일이 일어나겠죠..?

이 컨텍스트를 조작하려면 Thread Handle 이 있어야 해서 아까 Thread Create 이벤트에서 이 핸들을 저장해놨습니다.

어쨌거나 왜 이걸 안쓰기로 했냐면



```c++
if (softBreakPoint.count(ExceptionPos)) { //브레이크포인트 걸려있으면?
	success = RemoveBreakPoint(pi.hProcess,ExceptionPos);
	BackEIP(ThreadHandles[debug_event.dwThreadId]);
	setTrapFlag(ThreadHandles[debug_event.dwThreadId]);
}
```

0xCC를 복구하고, EIP를 -1하고, TrapFlag를 on 시키는데,

저 사이에 같은 쓰레드가 다시 생성되면서 0xCC를 다시 덮어씌워버리는 절묘한 상황이 발견되더라구요??

그래서 생각해보니 그냥 쓰레드 생성할때 TrapFlag를 활성화시키는 것으로 대체했습니다.

아마 제가 본격적으로 디버거를 만든다고 하면, 이렇게 모든 쓰레드에 Trap Flag를 활성화 시키고, BreakPoint는 그냥 내부적으로 해당 BP의 위치만 가지고서 거길 지나갈 때 실행을 잠시 멈추는 형태로 구현할 것 같습니다.



Trap Flag는 뭐냐 하면?

레지스터에 eflags가 있는데 이 Trap Flag가 활성화 되어있으면 다음 명령어에서 자동으로 코드 진행을 멈추고 디버거에게 Single-Step Event를 발생시킵니다. 이 Single-Step Event가 발생하면 TF는 0으로 초기화되구요.

Single Step 을 발생시키고 싶으실 때마다 활성화 시켜주시면 되겠습니다.

```c++
void setTrapFlag(HANDLE hThread) {
	CONTEXT lcContext;
	lcContext.ContextFlags = CONTEXT_ALL;
	GetThreadContext(hThread, &lcContext);
	lcContext.EFlags |= 0x100; //Active Trap Flag
	SetThreadContext(hThread, &lcContext);
}
```



이제 본격적으로 BreakPoint가 발생했을 때 어떤 처리를 하는지 확인해볼까요?



```c++
case EXCEPTION_SINGLE_STEP:
	ReadSomeCode(pi.hProcess, ExceptionPos, Codes, 16);
	insCnt = cs_disasm(cHandle, Codes, sizeof(Codes) - 1, ExceptionPos, 0, &insn);

	if (insCnt) {
		if (std::string(insn[0].mnemonic).find("call",0)==0) {
			printf("%d - 0x%" PRIx64 " : ", debug_event.dwThreadId, insn[0].address);
			for (int i = 0; i < ThreadCallDepth[debug_event.dwThreadId]; i++) {
				printf("  ");
			}
			printf("%s %s\n", insn[0].mnemonic, insn[0].op_str);
			ThreadCallDepth[debug_event.dwThreadId]++;
		}
		else if (std::string(insn[0].mnemonic).find("ret", 0) == 0) {
			printf("%d - 0x%" PRIx64 " : ", debug_event.dwThreadId, insn[0].address);
			for (int i = 0; i < ThreadCallDepth[debug_event.dwThreadId]; i++) {
				printf("  ");
			}
			printf("%s %s\n", insn[0].mnemonic, insn[0].op_str);
			ThreadCallDepth[debug_event.dwThreadId]--;
		}
	}
	if (ThreadHandles.count(debug_event.dwThreadId)) {  //핸들이 존재한다는거는 SoftwareBP를 걸었다는 뜻
		setTrapFlag(ThreadHandles[debug_event.dwThreadId]);  //TrapFlag는 리셋되므로 다시 설정
	}
	break;
```



ReadSomeCode 함수는 이렇게 생겼습니다.

```c++
bool ReadSomeCode(HANDLE hProcess, DWORD addr, BYTE *arr,size_t size) {
	DWORD dwReadBytes;
	return ReadProcessMemory(hProcess, (void*)addr, arr, size, &dwReadBytes);
}
```

특정 위치에 특정 바이트만큼 값을 읽는 함수입니다. 목표는 어셈블리 1개 명령어 이상을 메모리에서 값을 가져와야하는데, 왜 하필 16바이트냐 하면?

![](https://i.imgur.com/bWIshAY.png)

StackOverFlow에 찾아보니까 IA-32 개발자 메뉴얼에 따르면 15바이트가 최대라고 하네요.

물론 Repeating prefix를 이용하면 하나의 instruction 길이를 무한대로 늘릴 수 있다고 합니다.



저 cs_disasm함수는 유명한 디스어셈블 프레임워크인 capstone을 이용하는 부분입니다.  바이트 코드를 넣고 원하는 플랫폼, 아키텍처를 설정하면 디스어셈블 코드를 반환해줍니다.



이렇게 디스어셈블 한 코드중에서 명령어가 call이면 Depth를 +1해주고, ret이면 Depth를 -1 해줍니다,

Depth만큼 tab을 입력하고 명령어를 출력해주면? Call Trace를 화면에 표시할 수 있습니다.



![](https://i.imgur.com/L960ibX.gif)

(Wa! 유저영역 콜 스택! 작동영상: https://youtu.be/amkDWzhUmyk )



물론 함수를 호출하는 방법, 리턴하는 방법이 여러가지 방법이 있습니다.

호출할 때도, call 명령어 대신 push eip, jmp function 이런식으로 진행할 수도 있고,

ret도 ret명령어를 수행하는게 아니라 이 ret 명령어를 쪼개서 수동으로 복귀를 진행할 수도 있죠.

이러면 Call Trace가 꼬이게 되지만, 요즘 컴파일러 최적화를 진행하면 최대한 복합적인 어셈블리 명령어 하나를 써서 코드의 총 량을 줄이는게 프로그램의 실행 속도나 메모리를 최적화 하는데 도움이 되서 개발자가 직접 명령어를 수정하지 않는 한 문제가 생기진 않을.... 것 같긴 합니다..



다음에는 마이크로소프트에서 Symbol을 받아다가 어떤 함수를 호출하는지 이름을 출력할 수 있으면 출력하도록 해보겠습니다.



코드는.... 그냥 돌아가게만 만든 코드라서 리팩토링은 나중에 하겠습니다.

https://github.com/VertexToEdge/WindowFunctionTracer



# References

https://www.codeproject.com/Articles/43682/Writing-a-basic-Windows-debugger

https://www.codeproject.com/Articles/132742/Writing-Windows-Debugger-Part-2#DebugRunning

https://docs.microsoft.com/en-us/windows/win32/debug/debugging-functions

https://en.wikipedia.org/wiki/FLAGS_register