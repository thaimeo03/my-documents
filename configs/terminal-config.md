# PowerShell Configuration with Oh-My-Posh

## 1. Terminal Configuration
### Remove Logo
- Open JSON file and add `-nologo` to `commandline`.

### Appearance Settings
- **Font**: FiraCode Nerd Font (After installing Fira in NerdFonts)
- **Font Size**: 11 (After installing Fira NerdFont)
- **Background Opacity**: 92%

## 2. Autocomplete Configuration
Open an administrator terminal and run the following commands:

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted
Install-Module PSReadLine -Force
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle ListView
```

## 3. Set Up Oh-My-Posh
### Install Oh-My-Posh
```powershell
winget install JanDeDobbeleer.OhMyPosh -s winget
```

### Download and Install FiraCode Font
- Download 'FiraCode' from [NerdFonts](https://www.nerdfonts.com/font-downloads).
- Extract the downloaded file.
- Select all (`Ctrl + A`), right-click, and choose **Install**.

### Configure Oh-My-Posh
Open the terminal and run the following commands:

```powershell
oh-my-posh get shell
notepad $PROFILE
```

If you encounter an error, create the profile file with:

```powershell
New-Item -Path $PROFILE -Type File -Force
```

### Edit the Profile File
Copy the following command into the `.ps1` file and save it:

```powershell
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\amro.omp.json" | Invoke-Expression
```

### Final Step
- Reopen the terminal and test the configuration.

