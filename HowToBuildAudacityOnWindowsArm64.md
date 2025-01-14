# How to bulid audacity 3.17.0 on windows arm64?

## Prerequisites:
vs 2022
python 3.11 or later
git

## Commands:
Start cmd.exe, input the commands:
```
set VS_ROOT=<visual studio installation root>
mkdir build-uadacity
python -m venv build-env
.\build-env\Scripts\activate
pip install -U pip
pip install conan==2.11.0
"%VS_ROOT%\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsarm64.bat"
git clone https://github.com/audacity/audacity.git
cd audacity
git checkout Audacity-3.7.1
mkdir build
cd build
cmake .. -G "Visual Studio 17 2022" -A ARM64 -Thost=ARM64 -B output -DCMAKE_BUILD_TYPE=Debug
```

## Exist issues and solutions:
"Only Windows x64 supported" is displayed.

Find the conanfile.py of mpg123 in the conan2 cache (default is <User home>\.conan2), and fix it according to the following patch:

```
diff --git a/recipes/mpg123/all/conanfile.py b/recipes/mpg123/all/conanfile.py
index a4d19cd12463a..8a8094891085a 100644
--- a/recipes/mpg123/all/conanfile.py
+++ b/recipes/mpg123/all/conanfile.py
@@ -100,9 +100,10 @@ def build_requirements(self):
         if self.settings.arch in ["x86", "x86_64"]:
             self.tool_requires("yasm/1.3.0")
         if self._settings_build.os == "Windows":
-            self.win_bash = True
-            if not self.conf.get("tools.microsoft.bash:path", default=False, check_type=str):
-                self.tool_requires("msys2/cci.latest")
+            if is_msvc(self) == False:
+                self.win_bash = True
+                if not self.conf.get("tools.microsoft.bash:path", default=False, check_type=str):
+                    self.tool_requires("msys2/cci.latest")
 
     def source(self):
         get(self, **self.conan_data["sources"][self.version], strip_root=True)
```
Please refer to:
https://github.com/conan-io/conan-center-index/compare/master...codereba:conan-center-index:mpg123/removeMSYS2RequirementIfBuildWithMSVC

## After that run next commands:
cmake .. -G "Visual Studio 17 2022" -A ARM64 -Thost=ARM64 -B output -DCMAKE_BUILD_TYPE=Debug
cmake --build output
