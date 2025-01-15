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
cd build-uadacity
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

Find the conanfile.py of mpg123 in the conan2 cache (default is <User home>\.conan2), and fix it according to the following hilighed lines:

<table class="table table-bordered">
  <tbody>
    <tr>
      <code class="highlighter-rouge">
@@ -99,11 +99,10 @@ def build_requirements(self):
             self.tool_requires("pkgconf/2.0.3")
         if self.settings.arch in ["x86", "x86_64"]:
             self.tool_requires("yasm/1.3.0")
      </code>
    <tr>
      <em><strong>
      <code class="highlighter-rouge" color="blue">
       if self._settings_build.os == "Windows" and not is_msvc(self):
            self.win_bash = True
            if not self.conf.get("tools.microsoft.bash:path", default=False, check_type=str):
                self.tool_requires("msys2/cci.latest")
      </code>
      </strong>
      </em>
    </tr>
    <tr>
       <code class="highlighter-rouge">
     def source(self):
         get(self, **self.conan_data["sources"][self.version], strip_root=True)
       </code>
    </tr>
  </tbody>
</table>

Please refer to:
https://github.com/conan-io/conan-center-index/pull/26381

## After that run next commands:
```
cmake .. -G "Visual Studio 17 2022" -A ARM64 -Thost=ARM64 -B output -DCMAKE_BUILD_TYPE=Debug
cmake --build output
```

Audacity ARM64 will build ok.
