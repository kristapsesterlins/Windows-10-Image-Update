# Windows-10-Image-Update
Documentation about integrating and updating Windows 10 ISO

### References:
[Servicing The Image](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/servicing-the-image-with-windows-updates-sxs?view=windows-10)

[Mount and Modify a Windows image using DISM](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/mount-and-modify-a-windows-image-using-dism?view=windows-10)

## 1. Obtaing a Windows 10 ISO

[Media Creation Tool](https://www.microsoft.com/en-us/software-download/windows10ISO)

Press "Download Now" to download the Media Creation Tool

## 2. Copy the contents from the ISO media

Mount ISO (Right Click -> Mount) => Copy the contents of the ISO to a folder a hard drive.

## 3. Check the install.wim with Get-WindowsImage, to determine the Windows release.

```PS > Get-WindowsImage -ImagePath 'C:\Temp\Windows 10\Image\Win10_22H2_English_x64\sources\install.wim'```

```
ImageIndex       : 6
ImageName        : Windows 10 Pro
ImageDescription : Windows 10 Pro
ImageSize        : 15 529 007 739 bytes
```

## 4. Mount the install.wim

```PS > Mount-WindowsImage -ImagePath 'C:\Temp\Windows 10\Image\Win10_22H2_English_x64\sources\install.wim' -Index 6 -Path 'C:\Temp\Windows 10\Build\'```

## 5. Download the latest Windows 10 update for the specified release (22H2)

### Source:
[Windows 10 Update History](https://support.microsoft.com/en-us/topic/windows-10-update-history-8127c2c6-6edf-4fdf-8b9f-0f7be1ef3562)

_Be sure to read "Before installing this update" section to see if there are other updates missing_

### Example - 19045.2965 (KB5026361)
[KB5026361](https://www.catalog.update.microsoft.com/Search.aspx?q=KB5026361)

Choose => Windows 10, version 1903 and later and the needed CPU architecture => Download

## 6. Apply the downloaded image to the installation media

```PS > Add-WindowsPackage -Path 'C:\Temp\Windows 10\Build\' -PackagePath 'C:\Temp\Windows 10\Update\windows10.0-kb5026361-x64_961f439d6b20735f067af766e1813936bf76cb94.msu'```

## 7. Check the image if the update has been applied

```PS > Get-WindowsPackage -Path 'C:\Temp\Windows 10\Build\' | Sort-Object -Property InstallTime```

```
PackageName  : Package_for_ServicingStack_2905~31bf3856ad364e35~amd64~~19041.2905.1.0
PackageState : Installed
ReleaseType  : Update
InstallTime  : 15.05.2023 06:57:00

PackageName  : Microsoft-Windows-UserExperience-Desktop-Package~31bf3856ad364e35~amd64~~10.0.19041.2913
PackageState : Installed
ReleaseType  : OnDemandPack
InstallTime  : 15.05.2023 07:09:00

PackageName  : Microsoft-Windows-MediaPlayer-Package~31bf3856ad364e35~amd64~~10.0.19041.2965
PackageState : Installed
ReleaseType  : OnDemandPack
InstallTime  : 15.05.2023 07:09:00

PackageName  : Microsoft-Windows-QuickAssist-Package~31bf3856ad364e35~amd64~~10.0.19041.2913
PackageState : Installed
ReleaseType  : OnDemandPack
InstallTime  : 15.05.2023 07:09:00

PackageName  : Package_for_RollupFix~31bf3856ad364e35~amd64~~19041.2965.1.8
PackageState : Installed
ReleaseType  : SecurityUpdate
InstallTime  : 15.05.2023 07:09:00

PackageName  : Microsoft-Windows-Client-LanguagePack-Package~31bf3856ad364e35~amd64~en-US~10.0.19041.2965
PackageState : Installed
ReleaseType  : LanguagePack
InstallTime  : 15.05.2023 07:09:00
```

## 8. Reduce the size of the updated image

## Reference:
[Reduce the size of an image](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/mount-and-modify-a-windows-image-using-dism?view=windows-10#reduce-the-size-of-an-image)

```PS > dism /Image:'C:\Temp\Windows 10\Build' /Cleanup-Image /StartComponentCleanup /ResetBase```

## 9. Commit and save the change to the image

```PS > Save-WindowsImage -Path 'C:\Temp\Windows 10\Build\' -CheckIntegrity```

## 10. Unmount the image

```PS > Dismount-WindowsImage -Path 'C:\Temp\Windows 10\Build\' -Save```

# 11. Build a bootable ISO

## Source:
[ADK Install](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install)

When installing choose only the package "Deployment Tools".

The installation is available in "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Deployment Tools" directory.

```
PS > oscdimg -lCCCOMA_X64FRE_EN-US_DV9 -m -o -u2 -udfver102 -bootdata:2#p0,e,b"C:\Temp\Windows 10\Image\Win10_22H2_English_x64\boot\etfsboot.com"#pEF,e,b"C:\Temp\Windows 10\Image\Win10_22H2_English_x64\efi\microsoft\boot\efisys.bin" "C:\Temp\Windows 10\Image\Win10_22H2_English_x64" "C:\Temp\Windows 10\ISO\Win10_22H2_19045.2965_English_x64.iso"
```
