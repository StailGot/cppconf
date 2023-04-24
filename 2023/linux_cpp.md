---

title: "Обзор самых интересных проблем при портировании КОМПАС-3D под Linux"
author: "Александр Тулуп"
# theme : "serif"
# theme : "league"
theme : "night"
# theme : "black"
# highlightTheme : "default"
highlightTheme : "atom-one-dark"

transition : "slide"
logoImg: "img/logo-3.png"
controls: false
enableMenu: false
enableChalkboard: false
enableTitleFooter: false
---

<div style="text-align: left">

### Обзор самых интересных проблем при портировании КОМПАС-3D под Linux
<small>Александр Тулуп<br>tulup@ascon.ru</small>

<div style="line-height: 19pt ;font-size: 13pt">
<br><br><br><br><br><br><br>Конференция разработчиков<br>
АСКОН, Renga, C3D и Партнеров 2023</div>

</div>

---

### CMake

- `CMAKE_LINK_DEPENDS_NO_SHARED`
- `CMAKE_LINK_WHAT_YOU_USE`


---

#### CMAKE_LINK_DEPENDS_NO_SHARED

- Do not relink all depended targets on .so

---

### CMAKE_LINK_WHAT_YOU_USE

```bash { data-line-numbers }
Linking CXX shared library Build/Exe/.../libkHost.so
Warning: Unused direct dependencies:
	/Source/../Build/Exe/Debug-x64-Linux/libUI_Preview.so
	/Source/../Build/Exe/Debug-x64-Linux/libkDrawTool.so
	/Source/../Build/Exe/Debug-x64-Linux/libkSys.so
```

---

### MSVC

- `-fms-extensions`
- `-fvisibility-ms-compat`

---

### MSVC -fms-extensions

```c++ { data-line-numbers }
struct foo
{
  void baz() { std::cout << "Base" << std::endl; }
};

struct bar : public foo
{
  // Clang, -fms-extensions
  void baz() { return __super::baz(); }
};

int main()
{
  bar{}.baz();
}
```

---

### MSVC -fvisibility-ms-compat

```c++ { data-line-numbers }
// Foo.so
struct IBase
{
  virtual void Foo() = 0;
};
struct Foo : public IBase
{
  virtual void Foo() override {};
};
// Bar.so
int main()
{
 // -fvisibility-ms-compat, -stdlib=libc++
 ASSERT(dynamic_cast<Foo*>(GetBase()) != nullptr);
}
```

---

### MSVC -fvisibility-ms-compat

- `-fvisibility=hidden`
- `-stdlib=libc++`
- `__attribute__((visibility("default")))`

---

### MSVC -fvisibility-ms-compat

- `-Wweak-vtables`
- warning: A has no out-of-line virtual method definitions; its vtable will be emitted in every translation unit

---

### MSVC -fvisibility-ms-compat

- It sets the default visibility to hidden, like `-fvisibility=hidden`
- Types, but not their members, are not hidden by default.
- The One Definition Rule is relaxed for types without explicit visibility specifications that are defined in more than one shared object: those declarations are permitted if they are permitted when this option is not used.

---

### Link

- `-Wl,--no-undefined`
- `--exclude-libs,ALL`


---

### Link -Wl,--no-undefined

```c++ { data-line-numbers }
void Foo();

// -Wl,--no-undefined
// undefined reference to `Foo()'
void Bar()
{
  return Foo();
}
```

---

### Link --exclude-libs,ALL

- excludes symbols in all archive libraries from automatic export

---

### Warnings

- `-Werror`
- `-Wold-style-cast`
- `-Wgnu`
- `-Wunreachable-code`
- `-Wunused-function`
- `-Wdelete-abstract-non-virtual-dtor`
- `-Winfinite-recursion`
...

---

### ASAN

- `-fsanitize=address`

---

### Спасибо!

![link](img/link.png)

---

### Ссылки
- [-fvisibility-ms-compat](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html#index-fvisibility-ms-compat)
- [-fms-extensions](https://clang.llvm.org/docs/ClangCommandLineReference.html#cmdoption-clang-fms-extensions)
- [Run-Time Type Information (RTTI) ](https://itanium-cxx-abi.github.io/cxx-abi/abi.html#rtti)
- [Virtual Tables](https://itanium-cxx-abi.github.io/cxx-abi/abi.html#vague-vtable)
- [LINK_DEPENDS_NO_SHARED](https://cmake.org/cmake/help/latest/prop_tgt/LINK_DEPENDS_NO_SHARED.html#prop_tgt:LINK_DEPENDS_NO_SHARED)
- [LINK_WHAT_YOU_USE](https://cmake.org/cmake/help/latest/prop_tgt/LINK_WHAT_YOU_USE.html)