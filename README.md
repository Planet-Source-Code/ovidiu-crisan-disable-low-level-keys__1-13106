<div align="center">

## Disable Low Level Keys


</div>

### Description

There are many situations when it's need to disable some combinations of keys from a VB program. For instance, ALT-TAB, CTRL-ESC, ALT-ESC or others like these. Other combinations could be tested at form level using KeyPreview property and KeyPress / KeyDown / KeyUp events. All system keystrokes won't fire key events in a form (or other controls) because they are handled internally by the system. Since application threads never receive messages for these keystrokes, there is no way that an application can intercept them and prevent the normal processing. This behavior is "by design" and ensures that a user can always switch to another application’s window even if an application’s thread enters an infinite loop or hangs.

The question is how we can intercept this keystrokes? The solution could be achieved using hooks. A hook is a point in the Microsoft Windows message-handling mechanism where an application can install a subroutine to monitor the message traffic in the system and process certain types of messages before they reach the target window procedure.

For Windows NT SP3 (or higher), Microsoft introduced a new hook: WH_KEYBOARD_LL. This hook is called the low-level hook because it is notified of keystrokes just after the user enters them and before the system gets a chance to process them. This hook has a serious drawback: the thread processing the hook filter function could enter an infinite loop or hang. If this happens, then the system will no longer process keystrokes properly and the user will become incredibly frustrated. To alleviate this situation, Microsoft places a time limit on low-level hooks. When the system sends a notification to a low-level keyboard hook’s filter function, the system allows that function a fixed amount of time to execute. If the function does not return in the allotted time, the system ignores the hook filter function and processes the keystroke normally. The amount of time allowed (in milliseconds) is set via the LowLevelHooksTimeout value under the following registry subkey: HKEY_CURRENT_USER\Control Panel\Desktop.

The program (VB) is disabling some of these combinations (ALT-TAB, CTRL-ESC and ALT-ESC) as long as the option is checked.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Ovidiu Crisan](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/ovidiu-crisan.md)
**Level**          |Intermediate
**User Rating**    |4.3 (60 globes from 14 users)
**Compatibility**  |VB 5\.0, VB 6\.0
**Category**       |[Windows API Call/ Explanation](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/windows-api-call-explanation__1-39.md)
**World**          |[Visual Basic](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/visual-basic.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/ovidiu-crisan-disable-low-level-keys__1-13106/archive/master.zip)

### API Declarations

```
Option Explicit
Public Declare Sub CopyMemory Lib "kernel32" Alias "RtlMoveMemory" _
	(Destination As Any, Source As Any, ByVal Length As Long)
Public Declare Function GetKeyState Lib "user32" _
	(ByVal nVirtKey As Long) As Integer
Public Declare Function SetWindowsHookEx Lib "user32" Alias "SetWindowsHookExA" _
	(ByVal idHook As Long, ByVal lpfn As Long, ByVal hmod As Long, ByVal dwThreadId As Long) As Long
Public Declare Function CallNextHookEx Lib "user32" _
	(ByVal hHook As Long, ByVal nCode As Long, ByVal wParam As Long, lParam As Any) As Long
Public Declare Function UnhookWindowsHookEx Lib "user32" _
	(ByVal hHook As Long) As Long
Public Const HC_ACTION = 0
Public Const WM_KEYDOWN = &H100
Public Const WM_KEYUP = &H101
Public Const WM_SYSKEYDOWN = &H104
Public Const WM_SYSKEYUP = &H105
Public Const VK_TAB = &H9
Public Const VK_CONTROL = &H11
Public Const VK_ESCAPE = &H1B
Public Const WH_KEYBOARD_LL = 13
Public Const LLKHF_ALTDOWN = &H20
Public Type KBDLLHOOKSTRUCT
  vkCode As Long
  scanCode As Long
  flags As Long
  time As Long
  dwExtraInfo As Long
End Type
Dim p As KBDLLHOOKSTRUCT
Public Function LowLevelKeyboardProc(ByVal nCode As Long, ByVal wParam As Long, ByVal lParam As Long) As Long
  Dim fEatKeystroke As Boolean
  If (nCode = HC_ACTION) Then
   If wParam = WM_KEYDOWN Or wParam = WM_SYSKEYDOWN Or wParam = WM_KEYUP Or wParam = WM_SYSKEYUP Then
     CopyMemory p, ByVal lParam, Len(p)
     fEatKeystroke = _
      ((p.vkCode = VK_TAB) And ((p.flags And LLKHF_ALTDOWN) <> 0)) Or _
      ((p.vkCode = VK_ESCAPE) And ((p.flags And LLKHF_ALTDOWN) <> 0)) Or _
      ((p.vkCode = VK_ESCAPE) And ((GetKeyState(VK_CONTROL) And &H8000) <> 0))
    End If
  End If
  If fEatKeystroke Then
    LowLevelKeyboardProc = -1
  Else
    LowLevelKeyboardProc = CallNextHookEx(0, nCode, wParam, ByVal lParam)
  End If
End Function
```


### Source Code

```
Dim hhkLowLevelKybd As Long
Private Sub chkDisable_Click()
If chkDisable = vbChecked Then
  hhkLowLevelKybd = SetWindowsHookEx(WH_KEYBOARD_LL, AddressOf LowLevelKeyboardProc, App.hInstance, 0)
Else
  UnhookWindowsHookEx hhkLowLevelKybd
  hhkLowLevelKybd = 0
End If
End Sub
Private Sub Form_Unload(Cancel As Integer)
If hhkLowLevelKybd <> 0 Then UnhookWindowsHookEx hhkLowLevelKybd
End Sub
```

