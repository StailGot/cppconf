---
# https://github.com/evilz/vscode-reveal#-revealjs-options

title: Enum awesome 
description: "description"
author: "author"

theme : serif
# White, League, Sky, Beige, Simple, Serif, Blood, Night, Moon, Solarized

highlightTheme: xcode
# Default, A 11 Y Dark, A 11 Y Light, Agate, An Old Hope, Androidstudio, Arduino Light, Arta, Ascetic, Atelier Cave Dark, Atelier Cave Light, Atelier Dune Dark, Atelier Dune Light, Atelier Estuary Dark, Atelier Estuary Light, Atelier Forest Dark, Atelier Forest Light, Atelier Heath Dark, Atelier Heath Light, Atelier Lakeside Dark, Atelier Lakeside Light, Atelier Plateau Dark, Atelier Plateau Light, Atelier Savanna Dark, Atelier Savanna Light, Atelier Seaside Dark, Atelier Seaside Light, Atelier Sulphurpool Dark, Atelier Sulphurpool Light, Atom One Dark Reasonable, Atom One Dark, Atom One Light, Brown Paper, Codepen Embed, Color Brewer, Darcula, Dark, Darkula, Docco, Dracula, Far, Foundation, Github Gist, Github, Gml, Googlecode, Grayscale, Gruvbox Dark, Gruvbox Light, Hopscotch, Hybrid, Idea, Ir Black, Isbl Editor Dark, Isbl Editor Light, Kimbie Dark, Kimbie Light, Lightfair, Magula, Mono Blue, Monokai Sublime, Monokai, Nord, Obsidian, Ocean, Paraiso Dark, Paraiso Light, Pojoaque, Purebasic, Qtcreator Dark, Qtcreator Light, Railscasts, Rainbow, Routeros, School Book, Shades Of Purple, Solarized Dark, Solarized Light, Sunburst, Tomorrow Night Blue, Tomorrow Night Bright, Tomorrow Night Eighties, Tomorrow Night, Tomorrow, Vs, Vs 2015, Xcode, Xt 256, Zenburn

transition: slide
# Fade, Slide, Convex, Concave, Zoom

# logoImg: "https://raw.githubusercontent.com/evilz/vscode-reveal/master/images/logo-v2.png"
slideNumber: false

controls: true

help: true

separator: '^[\*]{3}[\s]*$'
verticalSeparator: '^[\-]{3}[\s]*$'

enableMenu        : false
enableChalkboard  : false
enableTitleFooter : false

enableZoom: true
enableSearch: true
---

# Clean code

## Awesome enum

<small>Created by [AleTu](http://AleTu.se) / [@AleTu](http://tulup@ascon)</small>

***

## Problem
```c++
    int main()
    {
      const auto fooId = foo.getId();

      if ( fooId != -1 )
      {

      }
    }
```

<span>
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="3" -->

<span>
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="3,5" -->

***

## Problem
```c++
    int main()
    {
      const auto fooId = foo.getThreadID(); // unsigned int
      const auto barId = bar.getShellID();  // unsigned int

      if ( fooId == barId )
      {
        ...
      }
    }
```

<span>
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="3" -->

<span>
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="3,4" -->

Wrong comparison
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="6" -->

***

## Problem
```c++
enum class ThreadID: std::uint32_t { undefined = SYS_MAX_UINT };
enum class ShellID: std::uint32_t { undefined = SYS_MAX_UINT };

int main()
{
  const auto fooId = foo.getThreadID(); // ThreadID
  const auto barId = bar.getShellID();  // ShellID

  if ( fooId == barId )
  {
    ...
  }
}
```

<span>
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="1" -->

<span>
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="1,2" -->

<span>
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="6" -->

<span>
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="6,7" -->

Error
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="6,7,9" -->

***

## Sample
```c++
    enum class MyAwesomeID: std::uint32_t
    {
      undefined = 0
    }

    int main()
    {
      auto fooId = foo.getId();

      if ( fooId != MyAwesomeID::undefined )
      {
        ...
      }
    }
```

<span class="fragment code-presenting-annotation current-only" data-code-focus="1">
  
</span>

<span class="fragment code-presenting-annotation current-only" data-code-focus="3">
  undefined -1, 0, MAX_UINT, ... 
</span>

<span class="fragment code-presenting-annotation current-only" data-code-focus="8">

</span>

<span class="fragment code-presenting-annotation current-only" data-code-focus="8, 10">

</span>


***

## underlying_type

```cpp
    enum class MyEnum: std::uint32_t;

    void foo()
    {
      using etype = std::underlying_type_t<MyEnum>;
      static_assert( std::is_same_v<etype, std::uint32_t> );
      ...
    }

```

***

## stl containers

```cpp
    enum class MyEnum: std::uint32_t;

    void foo()
    {
      std::unordered_map<MyEnum, std::string> map;
      std::set<MyEnum> set;
    }

```

***

## std::hash

```cpp
    enum class MyEnum: std::uint32_t;

    void foo()
    {
      EXPECT_EQ( std::hash<MyEnum>{}(MyEnum(42))
               , std::hash<std::uint32_t>{}(42) );
    }
```

***

## Init

```cpp
    enum class MyEnum: std::uint8_t;

    void foo()
    {
      MyEnum e1 = MyEnum(100500);
      MyEnum e2{100500};
    }
```

<span class="fragment code-presenting-annotation current-only" data-code-focus="5">
Before C++17
</span>
<span class="fragment code-presenting-annotation current-only" data-code-focus="6">
Since C++17
</span>
<span class="fragment code-presenting-annotation current-only" data-code-focus="1, 6">
Error: narrowing conversion
</span>

***

## Forward declaration
```c++

    enum MyAwesomeID;
    enum class MyAwesomeID: std::uint32_t;

    class IBar
    {
      ...

      virtual MyAwesomeID GetID() const = 0;
    };
```

Error before C++11 <br>
ISO C++ forbids forward references to 'enum' types
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="2" -->

Ok in MSVC <br> Achtung! Non std  extension
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="2" -->

Ok since C++11
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="3" -->

Ok since C++11 <br>Return by value
<!-- .element: class="fragment code-presenting-annotation current-only" data-code-focus="3,9" -->

<!-- ***

## CPP

<iframe width="1600px" height="500px" src="https://godbolt.org/e#g:!((g:!((g:!((h:codeEditor,i:(fontScale:1.9899450879999999,j:1,lang:c%2B%2B,source:'//+Type+your+code+here,+or+load+an+example.%0A%0A%23include+%3Ccstdint%3E%0A%23include+%3Ctype_traits%3E%0A%0Aenum+class+MyEnum:+std::uint32_t%3B%0A+%0A%0Aint+square(int+num)+%7B%0A++++static_assert(+std::is_same_v%3C%0A++++++++std::underlying_type_t%3CMyEnum%3E,+std::uint32_t%3E+)%3B%0A%7D'),l:'5',n:'0',o:'C%2B%2B+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((h:compiler,i:(compiler:g83,filters:(b:'0',binary:'1',commentOnly:'0',demangle:'0',directives:'0',execute:'1',intel:'0',libraryCode:'1',trim:'1'),lang:c%2B%2B,libs:!(),options:'--std%3Dc%2B%2B17',source:1),l:'5',n:'0',o:'x86-64+gcc+8.3+(Editor+%231,+Compiler+%231)+C%2B%2B',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0')),l:'2',n:'0',o:'',t:'0')),version:4"></iframe> -->

***

## To be continued ...

***
