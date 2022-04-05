# Lecture 5a: Ссылки

```cpp
Rational operator + (const int lhs, const Rational & rhs) {
    auto tmp = rhs;
    tmp.add(lhs);
    return tmp;
}
```

Обьект типа `Rational` . У автоматических переменных - автоматический тип размещения, а это значит, что  они удаляются при выходе из блока. То есть по сути `tmp` копируется. 

Что происходит с передачей параметров функций, что будет если мы уберем `&` у `rhs` .  Тогда мы просто скопируем их значения внутрь функции. Параметры для функции явлются полноценными локальными переменными, то есть это будут обьекты опять же с автоматическим типом размещения, и если бы у нас был более нагруженный пример, такая операция была бы не такой уж и дешевой. 

Что если мы просто хотим получить доступ к этим значениям? Для этого в языке есть такое понятие, как ссылка. (Это не та же ссылка, как в джаве). Мы с ссылками явно работаем. Передача по ссылке означает, что мы не копируем объект. Просто идентификатор `rhs` указывает на тот обьект, который был передан в эту функцию в качестве параметра. 

А можно ли такой фокус повторить с возвратом значения?

```cpp
Rational & operator + (const int lhs, const Rational & rhs)
```

Но мы так уже делали, когда говорили про операцию имплемента

```cpp
Rational & operator ++ () {
		numerator += denominator;
		return *this;
}
```

но например в постфиксном варианте - нету

```cpp
Rational operator ++ (int) {
		auto tmp = *this;
		numerator += denominator;
		return tmp;
}
```

ссылка лишь указывает на объект, в префиксной версии мы возвращаем ссылку на тот объект для которого оператор был вызвын, и с точки зрения возврата из функции времени жизни обьекта, оно охватывает момент вызова этой фунцкии. За пределами вызова этой функции объект существует, в случае с постфиксным операторм, мы создаем локальный объект и его возвращаем в качестве результата работы.

Если мы вернем ссылку

```cpp
Rational & operator ++ (int) {
		auto tmp = *this;
		numerator += denominator;
		return tmp;
}
```

то получится, что мы вернем ссылку на объект,который уже разрушен на момент завершения работы этой функции. Ссылка никуда не будет указывать - `undefined behavior` . 

Рассмотрим свободную функцию печати в поток

```cpp
std::ostream & operator << (std::ostream & strm, Rational x) {
    return x.print(strm);
}
```

Предположим, мы не хотели бы ничего делать для нашего типа. Вывод объекта нашего типа в поток должен был ничего не печатать. Мы можем

```cpp
return strm;
//return x.print(strm)
```

Здесь мы возврашаем ссылку на какой-то обьект, который мы передали. 

```cpp
public:
    const int & get_numerator() const {
        return numerator;
    }
```

Тут тоже все хорошо, поскольку объект, который мы возвращаем существует. Ссылка останется валидной.

Но в этом случае нету особого смысла возвращать по ссылке, поскольку оно является значением базового типа и ничуть не хуже будет его просто скопировать наружу, но об этом чуть позже

```cpp
int get_numerator() const {
		return numerator;
}
```

Обобщим: C точки зрения языка - ссылки - это нечто, что не является объектом, не является чем-то самостоятельным, это просто дополнительное имя к какому-то другому объекту. Всегда, когда работаем со ссылкой, мы работаем с идентификатором, и с точки зрения языка - это то же самое, что если бы в этом  месте стоял идентификатор нашего объекта. 

```cpp
int a = 10;
int & b = a;
++b;
```

Мы объявили ссылку на `а` и теперь у него два имени - `a` и `b` . И не важно через какое имя мы его промодифицируем. Изменится все тот же объект. У ссылки есть только область видимости и в рамках этой области видимости этой ссылкой можно пользоваться. 

Нельзя объявить ссылку без инициализации.

маленькое замечание:

```cpp
int & b = 10 // нельзя
const int & b = 10 // можно
// связвно это с литераломи, все очень хитро
```

```cpp
int a = 10;
int &b;
b = a;
++b;
```

То есть так нельзя, ошибка компиляции. Также ее нельзя переназначить, она всегда связана с единым обьектом. При этом нельзя пометить ссылку квалификатором `const` .

Если мы напишем

```cpp
const int & b = a
```

то это будет относиться к обьекту, к которому мы ссылаемся, через такую ссылку нельзя будет модифицировать наш обьект. 

Но есть, нюанс

```cpp
const int & b = a;
```

```cpp
const int a;
```

в первом случае, что мы сделали не означает, что сам обьект будет константным, его можно будет менять

```cpp
a++;
```

его нельзя будет менять через ссылку `b` .

Что будет, если мы вернем ссылку на объект, который разрушен?

```cpp
int & get_n(const int x) {
		int y = x;
		return y;
}
```

Современный компилятор нам выдаст предупреждение, но на самом деле у нас здесь `undefined behavior` . Все дело в том, что мы возвращаем ссылку на объект, время жизни которого короче, чем область видимости этой ссылки. 

А что если так?

```cpp
const int & get_n(const int & x) {
		return x;
}
```

Нам передали ссылку, мы ее возвращаем. тут все хорошо

```cpp
const int & b = get_n(a)
```

но что если мы напишем вот так?

```cpp
const int & b = get_n(10 + 3);
```

c точки зрения функции - все хорошо, но с точки зрения использования - не очень, поскольку мы передаем туда по ссылке временный объект, он будет существовать до конца полного выражения, где он был создан, и если мы попытаемся эту ссылку потом поиспользовать, то за пределами той инструкции выражения, наш объект не будет существовать, и у нас снова - `undefined behavior` . 

### cоnst

```cpp
const int & a; // 1
int const & b; // 2
```

 В конcтрукции `const` действует на тип `int` , то есть эта ссылка указывает на константный `int`. Обе эти конструкции эквивалентны между собой, но что важно, это то, что поставить спецификатор после амперсанта нельзя!  

Замечание 

```cpp
int const a = 0, b = 1;
```

В инструкции объявления могут быть сразу несколько переменных, тип будет один. 

Вообще еще есть псевдонимы типа, будет это выглядеть так

```cpp
using T = const int;
T a = 0, b = 1;
```

здесь тип `T` относится к всем переменным в объявлении.

Но ссылки могут быть разными. Это к категориям значения, но об этом говорить не будем, пока только коснемся того , какую роль это играет с ссылками

Но если кому интересно: [Категории выражений в C++ / Хабр (habr.com)](https://habr.com/ru/post/441742/?)

### lvаlue и rvаlue

по сути это две разновидности ссылок 

```cpp
int a = 0;
int & x = a; // lvalue ссылка
int && c = 5; // rvalue ссылка
// можем и так
const int & y = 10;
const int && u = 11;
```

мы ссылку `x` связали с объектом `а` , а остальные со временными объектами, литерал объектом не является, здесь будет вступать в силу правило языка - материализация. То есть, когда мы связываем некоторое значение с ссылкой, то происходит материализация, и значение превращается во временный обьект. 

Можно ли `y` связать с постоянным обьектом

```cpp
const int & y = a;
```

можно. Но можно ли со временным обьектом?

```cpp
const int & y = 10;
```

А можно ли то же самое сделать с `rvalue` ссылкой?

С постоянным

```cpp
int && z = a;
```

нельзя! `rvalue` ссылки можно связывать только со временными обьектами, либо, если в вход вступают операции приведения типов. Но вроде мы говорили, что временный объект должен быть разрушен, когда мы выходим за рамки этого выражения. А тут у нас исключение этого правила, время жизни временного обьекта продливается, если он связан с ссылкой. Продливается только на область видимости этой ссылки. 

Зачем нужны `rvalue` ссылки?

Вроде бы мы  прекрасно  связали временный обьект с константной value ссылкой, 

```cpp
const int & y = 10;
```

но можем ли мы с неконстантной `lvalue` ссылкой связвть обьект.

```cpp
int & x = 11;
```

оказывается, что нет! Все это произошло из-за того, что сначала были просто ссылки (`lvalue` ссылки), и неконстантные ссылки нельзя было связывать с временными объектами, а константные - можно было. Но потом все-таки захотелось связывать неконстантные ссылки с временными объектами. 

`rvalue` ссылки ввели только для того, чтобы можно было неконстантную ссылку связывать со временным объектом.

Предположим, у нас что-то такое

```cpp
const std::optional<Rational> & x = Rational::read(std::cin);
x += 5; // так не можем, но и сделать не const тоже не можем
// напоминаю что неконстантную lvalue ссылку нельзя связать с временным обьектом
```

но так мы можем

```cpp
std::optional<Rational> && x = Rational::read(std::cin);
x += 5;
```

но теперь можно задать вопрос, а зачем нужны константные `rvalue` ссылки, так вот - особо они не нужны, просто их наличие обусловлено тем, что правила про константность/неконстантность и `lvalue/rvalue` ссылки ортогональны. По сути, чтобы не было дополнительных исключений. 

Может ли быть ситуация, когда время жизни не продливается? 

На самом деле мы это уже где-то видели

```cpp
Rational operator + (const int lhs, const Rational & rhs) {
    auto tmp = rhs;
    tmp.add(lhs);
    return tmp;
}
```

Если мы возвращаем по ссылке и пытаемся вернуть временный объект, то его время жизни не продлевается, но на самом деле, это не является временным объектом, а является локальным объектом. 

Но мы можем тоже самое придумать  для временного объекта

```cpp
Rational & get_r(int a, int b) {
		return {a, b};
}
```

Здесь у нас объекты `Rational` внутри функции не существуеют, он создается внутри выражения инструкции `return` и он здесь является временным. И то что мы пытаемся вернуть по ссылке - никак не продливает его время жизни, и , получается, это будет классической висячой ссылкой.

```cpp
struct X {
    const int & n;
    
    X(const int &n_): n(n_) {}
};

int main() {
    X x(5 + 10);
}
```

 Если мы сделаем вот так вот, то полe `n` тоже окажется повисшей ссылкой, при этом если бы мы передали не временный объект, а постоянный, то все было бы ок

```cpp
int main() {
		int a;
		X x(a);
}
```

### Вернемся к разговору про классы

После того, как мы добавили права доступа, у нас перестала работать агрегатная инициализация. Это случилось потому что мы лишились публичного доступа к полям нашего класса.  Поэтому, мы можем сделать конструктор для класса, у одного класса могут быть несколько конструкторов, но об этом подробнее, когда будем говорить про перегрузки.

```cpp
struct Rational {
private:
    int numerator = 0;
    int denominator = 1;
public:
    Rational(const int a, const int b)
            : numerator(a) // список инициализации, это необязательная вещь, можно и 
            , denominator(b) { // и без нее
        
    }
		/* ... */
}
```

```cpp
Rational(const int a, const int b) {
		numerator = a;
		denominator = b;
}
```

Эквивалентны ли эти записи? Конструктор вызывается тогда, когда происходит инициализация класса, в первом случае, до момента выполнения тела конструктора осуществлена инициализация сначала всех базовых классов, затем всех полей нашего класса.  В более сложном примере у нас могли бы служить в качестве полей какие-то другие классы. И это правило про инициализацию всех полей до вызова конструктора  позволяет безопасно внутри тела конструктора использовать поля сложных типов. И поэтому удобен список инициализации, поскольку можно влиять на инициализацию полей. 

То есть если было бы так

```cpp
struct Rational {
private:
    int numerator;
    int denominator;
public:
    Rational(int a, int b) {
				numerator = a;
				denominator = b;
    }
```

Все было бы в целом корректно, но эти поля были уже проинициализированы в момент выполнения тела конструктора, при этом у них бы было неопределенное значение и строго говоря так нельзя делать, но опять же, для базовых типов это не очень большая проблема - Запишу в поле до или после конструктора, но для полей сложного типа не все так просто.

Также в списке инициализации можно иниц. базовые классы

```cpp
class X {
    int a;
public:
    X(const int a_): a(a_) {}
};

class Y : public X {
    double f;
public:  
    Y(const int x, const double y) 
        : X(x)
        , f(y) {
        
    }
};
```

Больше ничего в списке инициализации делать нельзя. 

Как со списком инициализации взаимодействует инициализация по умолчанию, которую мы раньше делали?

Она в некотором смысле заменяет список инициализации. Если мы указываем список инициализации, то он перекрывает инициализацию по умолчанию, у нас не будет двойной инициализации, у нас инициализация однократная, та, которая по умолчанию не будет рассматриваться компилятором.  

Но теперь ошибка вылезает тут

```cpp
std::optional<Rational> Rational::read(std::istream &strm) {
    Rational r;
    char c;
    strm >> r.numerator >>  c >>r.denominator;
    if (c == '/') {
        return r;
    }
    return {};
}
```

поэтому

```cpp
Rational() {

}
```

конструктор без аргументов - конструктор по умолчанию, если это добавим, то от ошибки избавимся, как только мы добавили какой-то конструктор у нас исчез конструктор по умолчанию (раньше был именно он).

 Компилятор может некоторые части кода сам генерить (этот конструктор). 

Но мы можем  и так (выше это то, что происходит, когда мы пишем `default` (c пустым телом)) 

```cpp
Rational() = default;
```

Теперь поговорим о немного другом, а ровно о противоположном, о деструкторе

### Деструктор

```cpp
~Rational() {

}
```

начинается с `~` , аргументы никакие не даются. 

Деструкторы используются в том случае, когда мы хотим освободить память. 

Если мы его не объявим, то компилятор его сам автоматически сгенерит. 

Рассмотрим жизненный цикл объекта сложного типа.

1. Выделяется под него память
2. Выполняется инициализация (конструктор), также и все поля класса, тело класса итд
3. Объект живет
4. Выполняется тело деструктора
5. Разрушаются поля класса в порядке обратном их инициализации
6. То же и происхдит с под-объектами базовых классов 

интересно еще то, что порядок, который мы записали в списке инициализации компилятор игноририует. Он использует стандартный порядок, то есть в каком порядке поля записаны были в классе, в том и инициализируются. 

Рассмотрим еще один маленький пример

```cpp
struct A {
    A(int, double) {
        
    }
};

struct B : A {
    
};
```

можно ли написать сразу же вот так

```cpp
B b(1, 05)
```

Нельзя, поскольку конструктора нет в классе `B` , но мы же унаследовались от класса `А` , в котором он есть. На самом деле он есть, но он спрятан.

можем починить, написав

```cpp
struct B : A {
		B(int , double b) : A(a, b) {
    }
};
```

но это как-то долго, не хочется так делать, и мы можем

```cpp
struct B : A {
		using A::A;    
};
```