# GPU Audio SDK - Windows Installation Notes

## Environment

- OS: Windows 11
- Visual Studio: 2022 Community (v143 toolset, MSVC 14.34.31933)
- CUDA: 12.9 at `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.9`
- CMake: 4.3+

## Working Configure Command

```powershell
cmake -S . -B "@BUILD" -G "Visual Studio 17 2022" -T "v143,version=14.34" -DWITH_HIP:BOOL=false -DCUDA_VERSION="12.9"
```

## Working Build Command

```powershell
cmake --build "@BUILD" --config RelWithDebInfo --parallel
```

## Lessons Learned

### 1. `@BUILD` is a literal directory name, not a placeholder
The cmake scripts check for the string `@BUILD` in file paths to identify generated files:
```cmake
if(source_file MATCHES ".*@BUILD.*")
```
Using any other build directory name (e.g. `build`) causes `source_group` errors during configure.

### 2. Quote `@BUILD` in PowerShell
PowerShell treats `@` as a special character (array syntax). Always quote it:
```powershell
cmake -S . -B "@BUILD" ...
cmake --build "@BUILD" ...
```

### 3. Pin the MSVC toolset version
CUDA 12.9 supports MSVC up to 14.34 (VS 2022 v143). Without `-T "v143,version=14.34"`, CMake picks the latest installed toolset which may be unsupported by nvcc.

### 4. CUDA registry keys may be missing after install
MSBuild's CUDA targets resolve `CudaToolkitDir` from:
`HKLM\SOFTWARE\NVIDIA Corporation\GPU Computing Toolkit\CUDA\v12.9` → `InstallDir`

If the CUDA installer ran without sufficient privileges, these keys won't exist. Fix with admin PowerShell:
```powershell
$cudaPath = "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.9\"
$regKey = "HKLM:\SOFTWARE\NVIDIA Corporation\GPU Computing Toolkit\CUDA\v12.9"
New-Item -Path $regKey -Force
New-ItemProperty -Path $regKey -Name "InstallDir" -Value $cudaPath -PropertyType String -Force
```

### 5. CUDA_PATH environment variable
Must be set for MSBuild to find the toolkit. Verify with `cmd /c "set CUDA"`.
If missing, set permanently:
```powershell
setx CUDA_PATH "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.9"
setx CUDA_PATH_V12_9 "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.9"
```
Then open a new terminal before running cmake.

### 6. CUDA Visual Studio integration files
Located at: `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.9\extras\visual_studio_integration\MSBuildExtensions\`
Must be present in: `C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Microsoft\VC\v170\BuildCustomizations\`
The CUDA installer handles this, but may need to be done manually if VS was installed after CUDA.
