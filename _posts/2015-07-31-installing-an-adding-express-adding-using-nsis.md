---
title:  "Installing an Add-in Express addin using NSIS"
date:   2015-07-31 13:06:00
categories: nsis add-in-express
---

> This post was migrated from my old WordPress installation and later from my Ghost installation and was initially published back in September 2013.

I'm currently in the end phase of a really nice project using Add-in Express to create plugins for Microsoft's Office suite. While it comes with support for generating a WIX-installer for you which works fine I always liked NSIS more and have good templates ready for my own branded installers.

Breaking the WIX-installerscript that was generated apart wasn't that hard thankfully. The important parts, i.e. registering the addins, was easy to extract and relies on the `adxregistrator.exe` and `AddinExpress.MSO.2005.dll` which is available from the Add-in Express installation folder. So by embedding those in my project and then calling `adxregistrator.exe` to do the actual registration made this simple enough. It's important to remember to keep `adxregistrator.exe` after the installation though since it's needed to uninstall the application afterwards.

**AddinInstaller.nsi**

{% highlight nsis %}
Section "Office Addins"
  ; To make this check you need the FindProc-plugin for NSIS and the
  ; macro defined below in the article.
  !insertmacro CheckAppRunning "OUTLOOK" "OUTLOOK.EXE" "Microsoft Outlook"
  !insertmacro CheckAppRunning "EXCEL" "EXCEL.EXE" "Microsoft Excel"
  !insertmacro CheckAppRunning "WORD" "WINWORD.EXE" "Microsoft Word"
  !insertmacro CheckAppRunning "POWERPOINT" "POWERPOINT.EXE" "Microsoft PowerPoint"
  
  SetOutPath "$INSTDIR"
  ; We need a few files from the Add-in Express folders, you might need
  ; to update the paths to match your system though.
  File "C:\Program Files (x86)\Add-in Express\Add-in Express for .NET\Redistributables\adxregistrator.exe"
  File "C:\Program Files (x86)\Add-in Express\Add-in Express for .NET\Bin\AddinExpress.MSO.2005.dll"
  
  ; We need all outputted files from the project, make sure that you have registered
  ; the project with Office at least once so that the adxloader-dlls are available
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\adxloader.dll"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\adxloader.dll.manifest"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\adxloader64.dll"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\Microsoft.Office.Interop.Excel.dll"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\Microsoft.Office.Interop.Outlook.dll"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\Microsoft.Office.Interop.PowerPoint.dll"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\Microsoft.Office.Interop.Word.dll"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\Microsoft.Vbe.Interop.dll"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\Office.dll"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\Shorthand.Core.OfficeAddins2.Addin.dll"
  File "..\Shorthand.Core.OfficeAddins2.Addin\bin\Release\Shorthand.Core.OfficeAddins2.Addin.tlb"
  
  ; Finally we call adxregistrator.exe with the path to our dll to register it on the target system
  ExecWait "$INSTDIR\adxregistrator.exe /install=$\"$INSTDIR\Shorthand.Core.OfficeAddins2.Addin.dll$\" /privileges=user /returnExitCode=false"
  SectionEnd
  
  Section Uninstall
  ; And of course, when we run the uninstaller we call adxregistrator.exe again to unregister the addin
  ExecWait "$INSTDIR\Office Addins\adxregistrator.exe /uninstall=$\"$INSTDIR\Shorthand.Core.OfficeAddins2.Addin.dll$\" /privileges=user"
SectionEnd
{% endhighlight %}

**CheckAppRunning.nsi**

{% highlight nsis %}
!macro CheckAppRunning ID Proc Name
  checkApp${ID}:
  FindProcDLL::FindProc "${Proc}"
  IntCmp $R0 1 0 notRunning${ID}
  MessageBox MB_OK|MB_ICONINFORMATION "${Name} is currently running and needs to be closed before the installation can continue."
  goto checkApp${ID}
  notRunning${ID}:
!macroend
{% endhighlight %}


The CheckAppRunning code is needed if you want to check if any of the office applications are running (might interfere with the installer). You will also need to install the [FindProc-plugin](http://nsis.sourceforge.net/FindProcDLL_plug-in) into your NSIS-plugins folder.