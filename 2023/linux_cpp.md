---

title: "Компиляция под Linux"
author: "Александр Тулуп"
# theme : "serif"
# theme : "league"
theme : "black"
highlightTheme : "default"

transition : "slide"
logoImg: "img/logo.png"
controls: false
enableMenu: false
enableChalkboard: false
enableTitleFooter: false
---

#### Компиляция под Linux
<small>Александр Тулуп<br>tulup@ascon.ru</small>

---

### CMake

- `CMAKE_LINK_DEPENDS_NO_SHARED`
- `CMAKE_LINK_WHAT_YOU_USE`

---

### ASAN

- `-fsanitize=address`


---

### MOLD

- `-fuse-ld=${MOLD_PATH}`


---

### Link

- `-Wl,--no-undefined`
- `--exclude-libs,ALL`


---

### MSVC

- `-fvisibility-ms-compat`
- `-fms-extensions`


---

### Спасибо!

---

### Ссылки
- [cmdoption-clang-fvisibility-ms-compat](https://clang.llvm.org/docs/ClangCommandLineReference.html#cmdoption-clang-fvisibility-ms-compat)
- [-fvisibility-ms-compat](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html)
- [cmdoption-clang-fms-extensions](https://clang.llvm.org/docs/ClangCommandLineReference.html#cmdoption-clang-fms-extensions)