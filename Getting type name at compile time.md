<!-- $theme: default -->
<!-- page_number: true -->
<!-- $size: 16:9 -->

# Getting type name at compile time
## Adam Badura, Nokia

---
<!-- footer: Getting type name at compile time, Adam Badura, Nokia, code::dive 2018 -->

# We have the function name

- `__func__` (C99, C++11)
- `__PRETTY_FUNCTION__` ([GCC](https://gcc.gnu.org/onlinedocs/gcc/Function-Names.html) & clang)
- `__FUNCSIG__` ([MSVC](https://docs.microsoft.com/en-us/cpp/preprocessor/predefined-macros))
- [`BOOST_CURRENT_FUNCTION`](https://www.boost.org/doc/libs/1_68_0/boost/current_function.hpp)

---

# What about the class name?

- [Is there a \_\_CLASS\_\_ macro in C++?](https://stackoverflow.com/questions/1666802/is-there-a-class-macro-in-c) - StackOverflow
- [Preprocessor macro to get the name of the current class? [duplicate]](https://stackoverflow.com/questions/5427099/preprocessor-macro-to-get-the-name-of-the-current-class) - StackOverflow
- [Is there a class name macro?](https://bytes.com/topic/c/answers/838227-there-class-name-macro) - Bytes
- [Is there a \_\_CLASS\_\_ macro in C++?](https://code.i-harness.com/en/q/196ef2) - CODE A&A Solved

---

# `type_name` to the rescue!

```c++
static_assert(type_name_v<int> == "int");
static_assert(type_name_v<decltype(0.1 * 10)> == "double");
```

---

# With the help of `__PRETTY_FUNCTION__` / `__FUNCSIG__`

```c++
template<typename T>
void foo()
{
	std::puts(__PRETTY_FUNCTION__); // for GCC & clang
	//std::puts(__FUNCSIG__); // for MSVC
}
```

- GCC: `void foo() [with T = {type}]`
- clang: `void foo() [T = {type}]`
- MSVC: `void __cdecl foo<{type}>(void)`

---

# Standard `__func__` is useless here

```c++
template<typename T>
void foo()
{
	std::puts(__func__);
}
```

- GCC: `foo`
- clang: `foo`
- MSVC: `foo`

---

# Underlying type

- All behave as if they were string literals.
- Much like `__FILE__`.
- However, those are not preprocessor symbols and will outlive preprocessing phase!

---

# `constexpr` and `std::string_view`

```c++
template<typename T>
constexpr auto foo()
{
	constexpr std::string_view full_name{ __PRETTY_FUNCTION__ };
	constexpr std::string_view left_marker{ "[with T = " };
	constexpr std::string_view right_marker{ "]" };

	constexpr auto left_marker_index = full_name.find(left_marker);
	static_assert(left_marker_index != std::string_view::npos);
	constexpr auto start_index = left_marker_index + left_marker.size();
	constexpr auto end_index = full_name.find(right_marker, left_marker_index);
	static_assert(end_index != std::string_view::npos);
	constexpr auto length = end_index - start_index;

	return full_name.substr(start_index, length);
}
```

---

# Tricky C-printing! :worried:

```c++
std::cout << type_name_v<int>;
```
or
```c++
constexpr auto name = type_name_v<int>;
std::printf("%.*s\n", static_cast<int>(name.size()), name.data());
```

---

# Unaware of aliases :disappointed:

```c++
static_assert(type_name_v<std::size_t> == "long unsigned int");
```

---

# Compiler dependent :disappointed:

- GCC
```c++
static_assert(type_name_v<std::string> == "std::__cxx11::basic_string<char>");
```
- clang
```c++
static_assert(type_name_v<std::string> == "std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >");
```

---

# ![](GitHub-Mark-64px.png) GitHub

https://github.com/adambadura/type_name

# Compiler Explorer

https://godbolt.org/z/vaPf7l

---

# Q&A