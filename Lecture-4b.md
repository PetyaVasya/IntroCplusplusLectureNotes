# Lecture 4b. Продолжение

### Статические члены классов

```cpp
struct Rational {
    int numerator = 0;
    int denominator = 1;

    void print(std::ostream & strm) {
        strm << numerator << "/" << denominator << "\n";
    }

    void add(int x);
    void add(Rational r);

    void divide(int x) {
        denominator *= x;
    }

    static std::pair<Rational, bool> read(std::istream & strm);

};

std::pair<Rational, bool> Rational::read(std::istream &strm) {
    Rational r;
    char c;
    strm >> r.numerator >>  c >>r.denominator;
    if (c == '/') {
        return {r, true};
    }
    return {{}, false};
}
```

мы определили статическую функцию класса, для того, чтобы считывать значения из потока ввода, но статическими могут быть не только функции, но и поля класса

```cpp
static int x;
```

Как в случае с статическими функциями, статические поля не связаны с конкретными обьектами нашего класса, а связаны с классом целиком, то есть, по сути, это глобальные сущности.  Выше мы обьявляем наше поле, но у него пока нет определения.  Мы можем к нему обращаться, использовать в качестве константы

Если наше поле не будет являться константным, то требуется определение. Определение этого поля всегда делается вне этого класса

```cpp
int Rational::x = 111;
```

Но можно объединить  обьявление и определение поля, используя ключевое слово `inline` 

```cpp
inline static int x = 1;
```

Для функций класса использовать `inline` не требуется. `inline` подразумевается. 

В 17 стандарте языка появился

```cpp
#include <optional>
```

Ему, в качестве шаблонного параметра, мы передаем только тип

```cpp
static std::optional<Rational> read(std::istream & strm);
```

Причем, обьект `optional` может иметь значение, а может его и не иметь. Наша функция теперь будет иметь такой вид

```cpp
std::optional<Rational> Rational::read(std::istream &strm) {
    Rational r;
    char c;
    strm >> r.numerator >> c >>r.denominator;
    if (c == '/') {
        return r;
    }
    return {};
}
```

теперь можно в мейне поприятнее написать

```cpp
int main() {
    auto r = Rational::read(std::cin);
    if (r) {
        r->print(std::cout);
    } else {
        std::cout << "wrong value\n";
    }
}
```

работает контекстное приведение к логичемкому типу, то есть мы можем просто поместить объект `optional` в данном случае в `if`; будет истинной, если значение находиться, если его нет -  то ложь. Чтобы допуститься  до самого значения, когда мы знаем, что оно есть, мы можем использовать оператор `->` , который нам еще будет нужен в указателях. Этот оперетор подобен оператору `.` . Несмотря на то что `optional` не указатель, он реализует в себе этот оператор.  

Также есть тип `Expected`

```cpp
static std::optional<std::string, Rational> read(std::istream & strm);
```

но этого пока что нету), в стандарт он еще не попал, но ничего не мешает взять  реализацию и подключить его к проекту. 

Напишем функцию сравнения

```cpp
bool compare(const Rational & x) {
		return numerator == x.numerator && denominator == x.denominator;
}
```

```cpp
int main() {
    auto r1 = Rational::read(std::cin);
    if (r1) {
        auto r2 = Rational::read(std::cin);
        if (r2) {
            std::cout << std::boolalpha << r1->compare(*r2) << "\n";
        } else {
            std::cout << "wrong value" << "\n";
        }
    } else {
        std::cout << "wrong value" << "\n";
    }
}
```

но, если прописать в мейне что объекты константные, мы не сможем сравнить их между собой, что очень странно.

```cpp
const auto r1 = Rational::read(std::cin);
```

если посмотреть на определение функции `compare` то она не меняет поля, мы бы хотели прописать это. Для этого к нашей функции можно добавить спецификатор `const`

```cpp
bool compare(const Rational & x) const {
		return numerator == x.numerator && denominator == x.denominator;
}
```

теперь можно вызывать от константных обьектов, но можно вызывать и не от константных. Теперь внутри нашей функции нельзя изменять поля обьекта, и компилятор это проверяет.  Даже нельзя вызывать неконстантные функции из нее, компилятор опять же не позволит.

У класса членом может быть не только функция, но и оператор

```cpp
bool operator ==(const Rational & x) const {
		return numerator == x.numerator && denominator == x.denominator;
}
```

тогда

```cpp
if (r2) {
		std::cout << std::boolalpha << (*r1 == *r2) << "\n";
} else {
		std::cout << "wrong value" << "\n";
}
```

можно также

```cpp
void operator /= (int x) {
		denominator *= x;
}
```

но написать что-то такое у нас сейчас не полулучится

```cpp
const auto r3 = (r1 /= 10)
```

```cpp
int a = 10;
int b = (a /= 5)++;
// a = 3, b = 2
```

по сути, этот оператор должен возврощать что-то, что указывает на наше значение `а`

```cpp
Rational operator /= (int x) {
		denominator *= x;
		return * this;
}
```

Давайте попробуем написать для нашего типа `++` 

```cpp
Rational & operator ++ () {
		numerator += denominator;
		return * this;
}
```

Но тут проблема в том, что `++` бывает разный, то есть `++a` отличается от `а++`. Интересно, какой написали мы? 

Так оказывается, мы написали префиксный. Как сделать постфиксный? 

Чтобы перегрузить постфиксный оператор, нам надо добавить псевдоаргумент, который должен иметь тип `int`, но мы его не будем использовать.

```cpp
Rational operator ++ (int) {
		auto tmp = * this
		numerator += denominator;
		return tmp;
}
```

Печать мы тоже можем оформить в виде оператора, но когда мы опредяем оператор как член класса, объектом класса всегда выступает первый аргумент оператора, а здесь нам нужен второй аргумент, мы не сможем сделать оператор членом этого класса, но это не проблема, мы вполне сможем перегрузить, как свободную функцию

```cpp
std::ostream & print(std::ostream & strm) const {
		return strm << numerator << "/" << denominator << "\n";
}
```

```cpp
std::ostream & operator << (std::ostream & strm, Rational x) {
    return x.print(strm);
}
```

Давайте еще сделаем оператор сложения

```cpp
Rational operator + (const Rational & x) const {
		auto tmp = *this;
		tmp.add(x);
		return tmp;
}
```

можно и сделать для других типов

```cpp
Rational operator + (const int & x) const {
		auto tmp = *this;
		tmp.add(x);
		return tmp;
}
```

если мы сделаем,

```cpp
r1 + 33.0 << "\n"
```

то  это сработает, поскольку произойдет неявное приведение типов.

```cpp
33 + r1 << "\n"
```

если мы напишем так,  компилятор кинет ошибку, у нас нет подходящего оператора. Когда мы определяем оператор как член класса, его первый аргумент всегда подразумевается объектом этого класса. 

```cpp
r1.operator  + 33 
```

вот что на самом деле происходит “*behind the curtains”.*  В прошлом случае надо определить оператор, как свободную функцию

```cpp
Rational operator + (const int lhs, const Rational & rhs) const {
    auto tmp = rhs;
    tmp.add(lhs);
    return tmp;
}
```

### Права доступа

На данный момент все поля нашего класса имеют публичные права доступа, это не очень, хорошо

```cpp
struct Rational {
private:
    int numerator = 0;
    int denominator = 1;
public:
    std::ostream & print(std::ostream & strm) const {
        return strm << numerator << "/" << denominator << "\n";
    }

    void add(int x);
    void add(Rational r);
    
    Rational operator /= (int x) {
        denominator *= x;
        return * this;
    }

    Rational operator ++ () {
        auto tmp = *this;
        numerator += denominator;
        return tmp;
    }

    bool operator ==(const Rational & x) const {
        return numerator == x.numerator && denominator == x.denominator;
    }

    static std::optional<Rational> read(std::istream & strm);
};
```

теперь вне из класса нельзя будет получить доступ к приватным полям. Что интересно, это что функция `read` является статическим членом класса, поэтому она имеет доступ к всем полям этого класса, вне зависимости от прав доступа, в свободных функциях так делать нельзя.

Бывают 3 разных типа прав доступа:

1. `private`  

это означает, что к этим членам имеет доступ только сам класс, нюанс только в том, что имеют еще доступ все вложенные классы.

1. `public`

Все имеют доступ к этим членам класса

1. `protected` 

Означает, что к данным полям может обратиться этот класс и все потомки этого класса

Чтобы определить класс, мы можем не только использовать ключевое слово `struct` , но и  `class` 

```cpp
class Rational {
		/* ... */
}
```

Разницы почти нет, но по умолчанию в `struct` все члены этого класса - публичные, а в классе - наоборот - приватные. 

Лучше использовать `struct` , когда он имеет только данные в качетсве полей (как `@dataclass` в пайтоне) 

Еще одна разница связана с наследованием

### Наследование

```cpp
struct X : Rational  {
		/* ... */  
};
```

В данном случае, класс `Х` наследуется от класса `Rational` . В С++ поддерживается множественное наслдедование, но об этом в будущем)

Что нам сейчас интересно, это то, что права доступа наследуются, при этом наследование  тоже бывает публичным, защищенным и приватным. По умолчанию, классы, которые задаются ключевым словом `struct` имеют публичное наследование, в `class` - наследование приватное, чтобы сделать публичным достаточно 

```cpp
class X : public Rational {
		/* ... */
}
```