---
title: C++ Reversing Series - 0x01
date: 2023-11-02
tags: [c++ reverse, c++ reversing, oop reverse, object oriented reverse, cpp reversing, c++ class reverse, rtti, rtti reversing]
description: We are with you with the second article of our series where we share tips that we can revise applications developed with C++ more easily.  
toc: true
lang: en
lang-ref: reversing-cpp-0x01
---

After a short break, we continue with the second article of our series in which we try to make the internal structures of C++ more clear to "Reverse Engineers". Thank you for your nice comments on the first article, it was nice and motivating to hear from you that such a series is needed. 

In this article, I will explain the RTTI (Run-Time Type Information) structure, which is one of the biggest features that C++ compilers (actually supported not only in C++ but also in several other languages) offer to the RE community, the advantages it offers us and how we can take advantage of it. But since it will be a bit long, I will make the part about RTTI in two parts. So in this article, we will introduce the topic a little more. 

## RTTI??

We briefly mentioned the advantages of an object-oriented language in our previous series. But what about the disadvantages? We know that classes are basically structs. That is, they are actually chunks of data that are arranged one after the other somewhere in memory according to their type. But what would happen if we wanted to evaluate an object at runtime other than its data type, size and the data contained in it? For example, let's imagine that we are working in a project with multiple data types and classes that are similar in size. In case of any adverse or unforeseen conditions, we use exceptions to do the best we can and we can pass it as a parameter to the catch block to direct it to the handler. But how can we evaluate by type in such a situation? Or how can we evaluate the relationship between inherited class objects at runtime? As a solution to these problems, the C++ ISO committee has proposed RTTI as a general solution. To understand the RTTI structure, there are a few important constructs we need to know first. 

### dynamic_cast<>()

Type casting is the general name given to the process of converting the type of a given structure to another type, and there are certain cast operators defined in C++ to achieve this. dynamic_cast is one of the different ones among these operators because it provides a conversion at runtime and can only be applied to **polymorphic** classes. In other words, there must be at least one virtual method in the class so that it can export the dynamic object type at runtime. 

While the static type casting operators (see: static_cast, reinterpret_cast) perform the conversion of one type to another at compile time, dynamic_cast does it at runtime and tells us the success of the type conversion in two ways. 

- If the object to be converted is compatible with the class to be converted and the conversion is successful, it returns the object pointer or reference given as **type_id**. 

- If the type conversion fails, it returns **null pointer** and throws **bad_cast** exception. 

### upcasting

It means that the pointer or reference of the Derived class object behaves like the Base class pointer. **Implicit** casting can be done, there is no need to cast explicitly because it is natural in OOP to do so.  

### downcasting

It means that a Base class object pointer or reference behaves like a Derived class pointer. **Explicit** casting is mandatory because it is not commonly used and it violates the nature of OOP due to the lack of **is-a** relationship. To summarize the relationships in a short way:

```cpp
class Root{};

class Plant{
    Root rootP;
};

class Tree : public Plant {};
```

#### is-a

According to the example above, we can say that a tree is a plant, i.e: "Tree <u>is a</u> Plant". The is-a relationship actually explains **Inheritance** in a simple way.

#### has-a

Plants have roots, i.e: "Plant <u>has a</u> Root"". The has-a relationship explains **Composition** in a simple way. 

Now that we have briefly summarized the concepts, let's be a little more descriptive by representing them in code.  

```cpp
#include <iostream>

class Message {
public:
    Message() {}
    virtual void getMessage() { std::cout << "Today it's very cold..."; };
};

class Share : public Message {
public:
    Share() {}
    void status() { std::cout << "Message shared successfully"; };
};

int main() {

    Message* msg = new Share;
    Share* twitter = dynamic_cast<Share*>(msg);

    twitter->status();

    return 0;
}
```

In the above class implementation, Message is our base class and Share is our derived class that takes the attributes of Message class. In the line `Message* msg = new Share;` we can implicitly assign our derived class object to our base class pointer and the operation we do here is called **upcasting**. In `Share* twitter = dynamic_cast<Share*>(msg);` the opposite is the case. We are trying to assign our base class pointer to our derived Share class pointer, so we are doing **downcasting**. And as you can see, we cast it explicitly with the **dynamic_cast** operator. Explicit casting is required by the compiler. Let's also emphasize that our base class is polymorphic when downcasting. Otherwise it would be a static object type, not dynamic. 

Hearing you say "*If downcasting is problematic and undesirable, then why do we use it?", I quote Scott Meyers' answer.

> "The need for **dynamic_cast** generally arises because we want perform **derived class operation** on a **derived class object**, but we have only a pointer-or reference-to-**base**." -Scott Meyers

### typeid & type_info

The typeid operator defined in the `typeinfo` header is actually useful for us in RTTI. Because with it, we can make some evaluations and determinations between dynamic types at runtime without writing complex classes or algorithms. 

The typeid operator basically returns the dynamic type of the object given to it as a parameter in the form of a type_info class reference. 

```cpp
class type_info {
public:
    type_info(const type_info& rhs) = delete; 
    virtual ~type_info();
    size_t hash_code() const;
    bool operator==(const type_info& rhs) const;
    type_info& operator=(const type_info& rhs) = delete; 
    bool operator!=(const type_info& rhs) const;
    int before(const type_info& rhs) const;
    size_t hash_code() const noexcept;
    const char* name() const;
    const char* raw_name() const;
};
```

As seen above, we can actually get a lot of information about dynamic objects thanks to the type_info class defined in the `typeinfo` header.   

```cpp
    std::cout << typeid(msg).raw_name() << "\n";
    std::cout << typeid(twitter).raw_name() << "\n";
    std::cout << typeid(*msg).raw_name() << "\n";
    std::cout << typeid(*twitter).raw_name() << "\n";
```

By adding on our existing code as above, we can get the **raw_name** attribute of both the object itself and the types of the objects it points to. As a result of this;

```
.PEAVMessage@@
.PEAVShare@@
.?AVShare@@
.?AVShare@@
```

(*Let's not worry about the meaningless suffixes before and after class names for now, I want to deal with name mangling separately later in this series*) 

Now we see the real benefit of RTTI. While we can't see the class names that we specify or write on a standard class, thanks to RTTI we can go into more detail and extract the classes from being classic structs. 

## Disassembler Perspective

After explaining the necessary terms with examples, let's compile our code with polymorphic class in **MSVC** compiler and take a look. 

```nasm
mov     [rsp+arg_8], rbx
push    rdi
sub     rsp, 30h
mov     ecx, 8          ; Size
call    ??2@YAPEAX_K@Z  ; operator new(unsigned __int64)
mov     rbx, rax
mov     [rsp+38h+arg_0], rax
lea     rax, ??_7Share@@6B@ ; const Share::`vftable'
mov     [rsp+38h+var_18], 0
lea     r9, ??_R0?AVShare@@@8 ; Share `RTTI Type Descriptor'
xor     edx, edx
lea     r8, ??_R0?AVMessage@@@8 ; Message `RTTI Type Descriptor'
mov     rcx, rbx
mov     [rbx], rax
call    __RTDynamicCast
```

When we look at the disassemble output, we can see the name of the classes we wrote (hereafter called object type). We need to pay attention to the vftable of the Share class. 

```nasm
dq offset ??_R4Share@@6B@  ; const Share::`RTTI Complete Object Locator'
; const Share::`vftable'
??_7Share@@6B@ dq offset sub_140001000
```

In the first article of our series, we talked about what vftable is, and here we can see that we already have 1 virtually defined function (*getMessage*). But what does the value named "**??_R4Share@@@6B@**" in the first line mean? Disassembler also gave us a clue by labeling it "**RTTI Complete Object Locator**", what is the purpose of this structure? 

I will answer these questions in a more advanced article on RTTI, see you soon...

## References

[Run-Time Type Information - Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/run-time-type-information?view=msvc-170)

[What Is Runtime Type Identification (RTTI) in C++? - CodeGuru](https://www.codeguru.com/cplusplus/what-is-runtime-type-identification-rtti-in-c/)

[Run-time type information - Wikipedia](https://en.wikipedia.org/wiki/Run-time_type_information)
