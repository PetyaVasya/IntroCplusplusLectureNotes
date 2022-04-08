# Lecture 4a. Сложные Типы

# **еnum**:

В мейне `side` - неквалифицированный идентификатор (если в имени нет`::`, то это оно и есть)

Можно ли сделать вот так?

```cpp
int main() {
    print(static_cast<Side>(1000));
}
```

```cpp
#include <iostream>

enum class Side { Buy , Sell };

void print(const Side side) {
    switch (side) {
        case Side::Buy:
            std::cout << "Buy";
            break;
        case Side::Sell:
            std::cout << "Sell";
            break;
    }
}

int side;

int main() {
    Side side = Side::Buy;
    print(side);
    std::cout << ::side << "\n";  
    /*тут мы обратились к глобальой перемене side
     * это выражение называется полностью квалифицированным 
     * иденфицирующим выражением, (путь от корня)
     * */	
}
```

Несмотрия на то что в enumе этого нет, нам можно это делать, но к сожалению ничего не напечатается, тут нельзя сказать, что код является “корректным”.

Можно ли наоборот?

```cpp
int main() {
    return Side::Sell;
}
```

А вот, компилятор выдал ошибку, он не может сделать такое неявное приведение типа,  а что если мы попросим его об этом явно?

```cpp
int main() {
    return static_cast<int>(Side::Sell);
}
```

Прекрасно, все получилось, код возврата 1. Явное приведение работает!

но в другую сторону явное приведение все же не работает

Еще в enumе можно задать конкретные значения вариантам типа, по умолчанию, компилятор делает это вот так:

```cpp
enum class Side { Buy = 0, Sell = 1}; // и.т.д
```

Но вообще, нас ничего не ограничивает и мы можем написать:

```cpp
enum class Side { Buy = 100, Sell = 200};
```

по умолчанию эти константы хранятся в интах, но мы также можем это изменить:

```cpp
enum class Side : unsigned long { Buy = 100, Sell = 200};
```

Данная конструкция есть только в плюсах, начиная с 11 стандарта. В языке С тоже есть enumы, но там конструкция выглядит немного иначе

```cpp
enum Side { Buy, Sell};
```

кстати, в плюсах еще можно писать так

```cpp
enum struct Side { Buy, Sell};
```

Но если полностью убрать `struct/class`, то у типа еnum будут немного другие свойства

```cpp
enum Side { Buy, Sell };
```

неявное приведение целочисленного типа в еnum все еще не работает, нам по прежнему нужно делать явное приведение, но, что интересно, в другую сторону теперь работает

```cpp
int main() {
    return Side::Sell; // работает неявное приведение
}
```

Даже так работает

```cpp
int main() {
    return Sell;
}
```

на самом деле - это не очень хорошо, пространство имен будет засаряться, если таких еnumов много ( конфликт имен )

В целом, лучше просто использовать механизм языка С++ и делать строгие еnumы. Если нужно провести преобразование к целочисленному типу, то это можно сделать явно. Если делать неявно, то можно не заметить ошибочное использование.

Немного натянутый пример, но все же

```cpp
#include <iostream>

enum Side { Buy, Sell };
enum Capacity { Agency, Principal, RisklessPrincipal };

void process(const Capacity capacity, const Side side) {
    if (side == Agency) {
        std::cout << "Well-well-well" << "\n";
    }
}

int main() {
    process(Principal, Buy);
}
```

Современный компилятор скажет, что Вы кринжанули, но это не будет  являться ошибкой, программа компилируется.

# struct:

Рассмотрим немного другой пример, давайте представим, что мы  работаем с числами, и нам очень важна точность вычислений. Так как мы знаем - числа с плавающей точкой бывают неточными, мы будем хранить числа так, как мы это делали в дни нашей прекрасной и светлой юности, то есть в школе - в виде дроби…

```cpp
struct Rational {
    int numerator;
    int denominator;
};
```

Это является полноценной инструкцией обьявления, а это значит, что мы сразу можем задать переменные

```cpp
struct Rational {
    int numerator;
    int denominator;
} a;
```

Допустим мы хотим обратиться к полям класса

```cpp
int main() {
    std::cout << a.numerator << "/" << a.denominator << "\n";
}
```

если мы сейчас запустим нашу программу, то она нам выдаст 0/0. Все дело в том как мы определили переменную а. Если мы сейчас сделаем так

```cpp
int main() {
    Rational a;
    std::cout << a.numerator << "/" << a.denominator << "\n";
}
```

то мы увидим мусор. Переменная была глобальная, а значит, у нее был статический тип размещения, а такие обьекты иницилизируется нулями по умолчанию. А в последнем случае такое не произошло, и там появился мусор.

Как нам проиницилизировать переменную данного типа? 1 вариант - это агрегатная инициализация

```cpp
int main() {
    Rational a{1, 2};
    std::cout << a.numerator << "/" << a.denominator << "\n";
}
```

А можно инициализировать по умолчанию

```cpp
struct Rational {
    int numerator = 0;
    int denominator = 1;
};
```

Мы можем свободно манипулировать нашими полями

```cpp
int main() {
    Rational a;
    a.numerator++; // 
    std::cout << a.numerator << "/" << a.denominator << "\n";
}
```

Теперь немного о другом, посмотрим, как у нас все это дело хранится в памяти

```cpp
int main() {
    Rational a;
    std::cout << sizeof(a) << "\n";
}
```

Вывод: 8 байт. Это не удивительно. В данном случае `int` имеет 4 байта, у нас 2 поля с типом `int`.

Усложним немного наш тип

```cpp
struct NewOrder {
    char side;
    double price;
    double volume;
};
```

`char` - 1 байт, `double` - 8. 

$$
2  \cdot 8 + 1 = 17 
$$

```cpp
int main() {
    std::cout << sizeof(NewOrder) << "\n";
}
```

А вот, наша программа выдала - 24 байт. Как будто у нас было 3 поля по 8 байт каждое. В С++ есть такое понятие, как выравинивание, оно связано с тем, что для процероссора намного удобнее доступаться к обьектам по одному машинному слову за раз. Желательно, чтобы обьекты в памяти были рассположены по адресам кратным машинному слову, если так не будет, то может возникнуть ситуация, в которой у нас какой-то обьект находится на границе двух машинных слов и - либо его не прочитают за раз, (за одну инструкцию обращения к памяти), либо могут возникнуть сложности, если оно окажется на границе двух страниц с точки зрения операционной системы, в общем - это сложно и так удобнее. 

Поэтому внутри сложного типа определяется поле с максимальным выравниванием, и все поля к нему приравниваются, что мы и увидили, все приравнялось к 8, и мы получили ответ - 24. Компилятор обеспечивает, что доступ к каждому полю будет выравненым

Если мы добавим еще одно поле `char`, то размер все равно будет 24

```cpp
struct NewOrder {
    char side;
    char a;
    double price;
    double volume;
};
```

Что делать если мы хотим это свойство обойти, на некоторых компиляторах можно применить такую конструкцию

```cpp
#pragma pack(1)
struct NewOrder {
    char side;
    char a;
    double price;
    double volume;
};
```

это позволило нам получить доступ к особенностям реализации языка. 

Чем мы жертвуем, если отключаем этот паддинг? 

На некоторых архитектурах, мы можем словить аппаратную ошибку. на x86, мы пожертвуем производительностью, но сэкономим немного памяти.

Тут, кстати, интересно

 

```cpp
struct NewOrder {
    char side;
    double price;
    char a;
    double volume;
    char b;
};
```

тут результат sizeоf будет 40, но, если мы переупорядочим наши поля 

```cpp
struct NewOrder {
    double price;
    double volume;
    char side;
    char a;
    char b;
};
```

здесь результатом будет 24.

Вернемся к нашему примеру, введем какие-то операции, 

```cpp
struct Rational {
    int numerator = 0;
    int denominator = 1;
    
    void print(std::ostream & strm) {
        strm << numerator << "/" << denominator;
    }
    
    void add(int x) {
        numerator += x * denominator;
    }
    
    void divide(int x) {
        denominator *= x;
    }
};
```

достаточно примитивная реализация, но тут есть несколько моментов

Внутри функций, которые мы определили в рамках класса можно обращаться к полям по их именам, то есть не квалифицированно, но тут мы сталкиваемся с усложнением области видимости.

```cpp
struct Rational {
    
    void print(std::ostream & strm) {
        strm << numerator << "/" << denominator;
    }
    
    void add(int x) {
        numerator += x * denominator;
    }
    
    void divide(int x) {
        denominator *= x;
    }

		int numerator = 0;
    int denominator = 1;
};
```

Если мы напишем так, то, конечно, ничего не изменится, но требуется некое пояснение. Дело в том, что область видимости имен - на уровне самого класса. В рамках всех других членов этого класса эта область видимости действует в независимости от места определения. 

```cpp
int x = denominator;
int numerator = 0;
int denominator = 1;
```

но так, например, нельзя, поскольку имя denominator еще не видно

```cpp
struct Rational {
    int numerator = 0;
    int denominator = 1;

    void print(std::ostream & strm) {
        strm << numerator << "/" << denominator;
    }

    void add(int x);
    
    void divide(int x) {
        denominator *= x;
    }
};

void Rational::add(int x) {
    numerator += x * denominator;
}
```

можно и выносить определения наружу, но внутри класса надо оставить обьявления этой функции

```cpp
struct Rational {
    int numerator = 0;
    int denominator = 1;

    void print(std::ostream & strm) {
        strm << numerator << "/" << denominator;
    }

    void add(int x);

    void add(Rational r);

    void divide(int x) {
        denominator *= x;
    }
};

void Rational::add(int x) {
    numerator += x * denominator;
}

void Rational::add(const Rational r) {
    numerator += r.numerator * denominator;
    denominator *= r.denominator;
    numerator *= r.denominator;
}
```

Кстати, в обьявлении функции мы можем опустить `cоnst` кваливификаторы верхнего уровня.

```cpp
#include <iostream>
#include <utility>

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
};

void Rational::add(int x) {
    numerator += x * denominator;
}

void Rational::add(const Rational r) {
    numerator += r.numerator * denominator;
    denominator *= r.denominator;
    numerator *= r.denominator;
}

std::pair<Rational, bool> read_rational(std::istream & strm) {
    Rational r;
    char c;
    strm >> r.numerator >>  c >>r.denominator;
    if (c == '/') {
        return {r, true};
    }
    return {{}, false};
}

struct NewOrder {
    double price;
    double volume;
    char side;
    char a;
    char b;
};

int main() {
    Rational a, b;
    auto res = read_rational(std::cin);
    if (res.second) {
        res.first.print(std::cout);
    } else {
        std::cout << "wrong value" << "\n";
    }
}
```

тут, кстати, удобно использовать ключевое слово `auto`. Начиная с 17ого стандарта мы можем

```cpp
auto [r, ok] = read_rational(std::cin);
if (ok) {
    r.print(std::cout);
} else {
    std::cout << "Wrong value" << "\n";
}
```

это синтаксический сахар, на самом деле только одна настоящая переменная, а эти имена связаны с ним.

### статические поля и статические функции класса

Обьявляем нашу функцию внутри класса

```cpp
static std::pair<Rational, bool> read(std::istream & strm);
```

в определении функции вне класса ключевое слово `static` можно упускать

Можно ли вызвать статическую функцию через обьект класса

```cpp
auto r2 = r.read(std::cin);
```

можно, но функция по прежнему вызывается вне зависимости от какого-то объекта нашего класса. Просто мы к ее имени допустились через имя обьекта класса. Чаще всего нет смысла так делать.

Итого, нестатические функции класса вызываются всегда относительно какого-то конкретного обьекта этого класса, статические вне зависимости, отличаются только именем и особенностями прав доступа.

Как происходит то, что нестатическая функция класса вызывается относительно конкретного объекта? 

На самом деле за кулисами компилятор будто бы передает функции дополнительный параметр. Этот параметр - указатель на обьект этого класса, и эта функция неявно его использует 

```cpp
void print(std::ostream & strm) {
    strm << this->numerator << "/" << this->denominator << "\n";
}
```

Иногда компилятор не знает как найти имя и приходится  так делать.
