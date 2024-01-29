---
title: C++ Reversing Serisi - 0x01
date: 2023-11-02
tags: [c++ reverse, c++ reversing, oop reverse, object oriented reverse, cpp reversing, c++ class reverse, rtti, rtti reversing]
description: C++ ile geliştirilen uygulamaları daha kolay reverseleyebileceğimiz ipuçları paylaştığımız serimizin ikinci makalesi ile sizlerleyiz. 
toc: true
---

C++'ın iç yapılarını "Tersine Mühendisler" nezdinde daha açık bir hale getirmeye çalıştığımız serimizin ikinci makalesine kısa bir aradan sonra devam ediyoruz. İlk makale hakkındaki güzel yorumlarınız için de ayrıca teşekkür ederim, böyle bir seriye ihtiyaç olduğunu sizlerden duymak güzel ve motive ediciydi. 

Bu makalemizde C++ compilerlarının (aslında sadece C++ nezdinde değil, bir kaç farklı dilde de destekleniyor) RE camiasına sunduğu en büyük özelliklerden olan RTTI (Run-Time Type Information) yapısını, bize sunduğu avantajları ve nasıl yararlanabileceğimizi anlatacağım. Fakat biraz uzun bir konu olacağından dolayı RTTI ile alakalı olan kısmı iki part yapacağım. Yani bu makalede biraz daha konuya giriş yapacağız. 

## RTTI??

Obje tabanlı bir dilin bize sunduğu avantajlardan kısaca bir önceki serimizde bahsetmiştik. Peki bize dezavantaj oluşturduğu durumlar? Classların temelde structlar olduğunu biliyoruz. Yani aslında tip türüne göre bellekte bir yerlerde ardı ardına dizilmiş veri yığınlarıdırlar. Fakat veri tipi, boyutu ve içerisinde bulunan datanın haricinde bir objeyi runtime anında değerlendirmek isteseydik ne olurdu? Mesela içerisinde birden fazla veri tipi ve boyut olarak benzeyen sınıfların olduğu bir projede çalıştığımızı düşünelim. Herhangi olumsuz veya öngörülemeyen bir koşulda elimizden gelenin en iyisini yapmak için exceptionları kullanırız ve handlera yönlendirmek için catch bloğuna parametre olarak geçebiliriz. Fakat böylesi bir durumda tipe göre değerlendirme nasıl yapabiliriz? Veya inherit edilmiş sınıf objeleri üzerindeki ilişkiyi runtime anında nasıl değerlendirebiliriz? Bu sorunlara çözüm olarak C++ ISO komitesi tarafından genel bir çözüm olarak RTTI sunulmuştur. RTTI yapısını anlamak için öncelikle bilmemiz gereken bir kaç önemli yapı var. 

### dynamic_cast<>()

Type casting, belirli bir yapının tipini başka bir tipe dönüştürme işlemine verilen genel isimdir ve bunu sağlamak için C++'da tanımlı belirli cast operatörleri vardır. dynamic_cast bu operatörler arasında farklı olanlardan birisidir çünkü runtime anında bir conversion sağlar ve sadece **polymorphic** classlara uygulanabilir. Yani classın içerisinde en az bir tane virtual method olması gereklidir ki runtime anında dinamik obje tipini export edebilsin. 

Statik type casting operatörleri (bknz: static_cast, reinterpret_cast) bir tipin diğer bir tipe dönüşümünü compile aşamasında sağlarken, dynamic_cast bunu runtime anında sağlar ve bize tip dönüşümünün başarı durumunu da iki şekilde belirtir. 

- Dönüştürülmek istenen obje, dönüştürüleceği class ile uyumlu ise ve başarılı bir şekilde dönüşüm yapılabiliyorsa **type_id** olarak verilen obje pointer veya referansını döndürür. 

- Tip dönüşümü başarısız olursa **null pointer** döndürür ve **bad_cast** exception fırlatır. 

### upcasting

Derived class objesinin pointer veya referansının Base class pointerı gibi davranması anlamına gelmektedir. **Implicit** (örtük) casting yapılabilir, açık bir şekilde cast etmeye gerek yoktur çünkü böyle bir işlemin olması OOP yapısında doğal birşeydir.  

### downcasting

Base class obje pointer veya referansının Derived class pointerı gibi davranması anlamına gelmektedir. **Explicit** (açık) casting yapılması zorunludur çünkü genellikle çok kullanılan bir işlem değildir ve **is-a** ilişkisinin olmamasından dolayı OOP doğasına aykırılık söz konusudur. Kısa bir şekilde ilişkileri de özetlemek gerekirse:

```cpp
class Root{};

class Plant{
    Root rootP;
};

class Tree : public Plant {};
```

#### is-a

Yukarıdaki örneğe göre ağaç bir bitkidir diyebiliyoruz, yani: "Tree <u>is a</u> Plant". Is-a ilişkisi aslında bize sade bir şekilde **Inheritance'ı** açıklar.

#### has-a

Bitkiler köklere sahiptirler, yani: "Plant <u>has a</u> Root"". Has-a ilişkisi ise bize sade bir şekilde **Composition'ı** açıklar. 

Kavramları kısa bir şekilde özetlediğimize göre kod üzerinden gösterimini yaparak biraz daha açıklayıcı olalım. 

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

Yukarıdaki class implementasyonunda, Message bizim base classımız, Share ise Message classının niteliklerini alan derived classımız olmaktadır. `Message* msg = new Share;` satırında implicit (örtük) bir şekilde derived class objemizi, base class pointerımıza atayabiliyoruz ve burada yaptığımız işlem **upcasting** adını alıyor. `Share* twitter = dynamic_cast<Share*>(msg);` satırında ise durum tam tersi. Base class pointerımızı kendisinden türetilen derived Share class pointerımıza atamaya çalışıyoruz, yani **downcasting** yapıyoruz. Ve görebileceğiniz üzere bunu explicit bir şekilde **dynamic_cast** operatörü ile cast ediyoruz. Explicit olarak cast etmek compiler tarafından zorunlu tutuluyor. Downcasting yaparken base classımızın polymorphic olduğunun da altını çizelim. Aksi takdirde dinamik değil, statik bir obje tipi olacaktı. 

"*Downcasting madem ki problemli ve istenmeyen bir yöntem, o zaman neden kullanıyoruz?*" dediğinizi duyarak Scott Meyers'ın cevabını alıntılıyorum.

> "The need for **dynamic_cast** generally arises because we want perform **derived class operation** on a **derived class object**, but we have only a pointer-or reference-to-**base**." -Scott Meyers

### typeid & type_info

`typeinfo` headerı içerisinde tanımlı olan typeid operatörü aslında bizim RTTI konusunda biraz elimiz ayağımız konumunda. Çünkü onunla birlikte kompleks classlar veya algoritmalar yazmadan runtime anında dinamik tipler arasında bazı değerlendirmeleri ve tespitleri yapabiliyoruz. 

typeid operatörü temel olarak kendisine parametre olarak verilen objenin dinamik tipini bize type_info class referansı şeklinde döndürür. 

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

Yukarıda da görüldüğü üzere `typeinfo`headerında tanımlı olan type_info classı sayesinde dinamik objeler hakkında aslında bir çok bilgiyi elde edebiliyoruz.  

```cpp
    std::cout << typeid(msg).raw_name() << "\n";
    std::cout << typeid(twitter).raw_name() << "\n";
    std::cout << typeid(*msg).raw_name() << "\n";
    std::cout << typeid(*twitter).raw_name() << "\n";
```

Mevcut kodumuz üzerine yukarıdaki gibi ekleme yaparak her iki objenin kendisinin ve point ettiği objelerin tiplerinin **raw_name** niteliğini alabiliriz. Bunun sonucunda da;

```
.PEAVMessage@@
.PEAVShare@@
.?AVShare@@
.?AVShare@@
```

çıktılarını almaktayız. (*Class isimlerinin önündeki ve sonundaki anlamsız eklere şimdilik takılmayalım, name mangling konusunu bu serinin devamında ayrı olarak ele almak istiyorum.*) 

İşte RTTI'ın gerçek faydasını şimdi görüyoruz. Standart bir class üzerinde bizim belirttiğimiz veya yazılan class isimlerini göremiyorken, RTTI sayesinde daha detaya inip classları klasik struct olmak çıkarabiliyoruz. 

## Disassembler Penceresinden Bakış

Gerekli terimleri örneklerle açıkladıktan sonra, kendi yazdığımız polymorphic classa sahip kodumuzu **MSVC** compilerında derleyip bir göz atalım. 

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

Disassemble çıktısına baktığımızda kendi yazdığımız classların adını (bundan sonra obje tipi olarak adlandırılacak) görebiliyoruz. Share classının vftable tablosuna dikkatimizi vermemiz gerekiyor. 

```nasm
dq offset ??_R4Share@@6B@  ; const Share::`RTTI Complete Object Locator'
; const Share::`vftable'
??_7Share@@6B@ dq offset sub_140001000
```

Serimizin ilk makalesinde vftable nedir kısmına değinmiştik, burada da zaten 1 adet virtual olarak tanımlanmış fonksiyonumuz (*getMessage*) olduğunu görebiliyoruz. Fakat ilk satırda yer alan "**??_R4Share@@6B@**" adına sahip değer ne anlama geliyor? Disassembler da bize "**RTTI Complete Object Locator**" etiketlemesini yaparak ipucu vermiş, acaba bu yapının amacı ne? 

Bu soruların yanıtını RTTI'ı daha ileri seviye ele alacağım makalede yayınlayacağım, görüşmek üzere...

## Referanslar

[Run-Time Type Information - Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/run-time-type-information?view=msvc-170)

[What Is Runtime Type Identification (RTTI) in C++? - CodeGuru](https://www.codeguru.com/cplusplus/what-is-runtime-type-identification-rtti-in-c/)

[Run-time type information - Wikipedia](https://en.wikipedia.org/wiki/Run-time_type_information)
