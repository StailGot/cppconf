---

title: "Обзор самых интересных проблем при портировании КОМПАС-3D под Linux"
author: "Александр Тулуп"
# theme : "serif"
# theme : "league"
theme : "black"
highlightTheme : "default"

transition : "slide"
logoImg: "img/logo-3.png"
controls: false
enableMenu: false
enableChalkboard: false
enableTitleFooter: false
---

#### Обзор самых интересных проблем при портировании КОМПАС-3D под Linux
<small>Александр Тулуп<br>tulup@ascon.ru</small>

<span style="font-size: 15pt">
<br>Конференция разработчиков<br>
АСКОН, Renga, C3D и Партнеров 2023</span>

---

### MSVC

- `-fms-extensions`
- `-fvisibility-ms-compat`

---

### Link

- `-Wl,--no-undefined`
- `--exclude-libs,ALL`

---

### CMake

- `CMAKE_LINK_DEPENDS_NO_SHARED`
- `CMAKE_LINK_WHAT_YOU_USE`

---

### ASAN

- `-fsanitize=address`


---

### MOLD

- `-fuse-ld=mold`

---

### Спасибо!

![link](img/link.png)

---

### Ссылки
- [-fvisibility-ms-compat](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html)
- [-fms-extensions](https://clang.llvm.org/docs/ClangCommandLineReference.html#cmdoption-clang-fms-extensions)
- [LINK_DEPENDS_NO_SHARED](https://cmake.org/cmake/help/latest/prop_tgt/LINK_DEPENDS_NO_SHARED.html#prop_tgt:LINK_DEPENDS_NO_SHARED)
- [LINK_WHAT_YOU_USE](https://cmake.org/cmake/help/latest/prop_tgt/LINK_WHAT_YOU_USE.html)