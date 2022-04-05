# Lecture 6b:

### `reinterpret_cast`

это вариант приведения типов на равне с операторами `static_cast` , `const_cast` , но в отличии от перечисленных он не выражается в машинных инструкциях. Это значит, что это просто инструкция для компилятора как смотреть на какой-то объект, по другому проинтерпретировать его объектное представление 

Что с ним можно делать? 

```cpp
const int x = reinterpret_cast<int>(10); 
```

Можно указатель привести к целочисленному типу 

```cpp
const auto y = reinterpret_cast<std::uintptr_t>(&x);
```

Реализация языка гарантирует, что значение указателя приведенному `std::uintptr_t` не потеряет в точности. Можно сделать и обратно и из целочисленного типа получить исходное значение на указателе. 

Что интересно, что если все сделать наоборот мы не факт что получим исходное число:

```cpp
std::uintptr_t i_ptr = 101;
char * ptr = reinterpret_cast<char *>(i_ptr);
const auto i_ptr2 = reinterpret_cast<std::uintptr_t>(ptr);
std::cout << std::boolalpha << (i_ptr == i_ptr2) << "\n";
```

Можно также привести из одного типа к ссылке на этот тип

```cpp
int i = -1;
unsigned & u = reinterpret_cast<unsigned &>(i);
std::cout << u << "\n";
```

Можно еще приходить из указателя на один тип на указатель на другой тип, но тут есть множество ограничений. Это можно делать безопасно приходя к типу `char` , `unsigned char` , `std::byte` . К другим не безопасно.

Можно приводить указатель на функцию к любому другому указателю на функцию, при этом если мы так сделаем, то вызывать через указатель нашу исходную функцию нельзя будет. Но можно привести обратно

Предположим у нас нету в языке полиморфизма, как в ооп, и нет шаблонов, но мы хотим написать один и тот же алгоритм для разных типов, как это можно делать?

 

```cpp
int a[] = {4, 5, 6, 3, 5}
std::qsort(a, sizeof(a) / sizeof(a[0]), sizeof(a[0]), cmp);
for (const auto x : a) {
		std::cout << x << " ";
}
```

`qsort` сортирует некоторый массив, мы передаем указатель на массив, количество элементов, размер элемента и компаратор

```cpp
int cmp(const void *lhs, const void *rhs) {
    int a = *static_cast<const int *>(lhs);
    int b = *static_cast<const int *>(rhs);
    if (a < b) return -1;
    if (a > b) return 1;
    return 0;
}
```

### Работа с памятью

Если мы захотим иметь динамическую СД, то есть что-то размер чего мы не знаем на этапе компиляции.  Для этого в языке есть два операра - `new` и `delete` . `new` память выделяет, `delete` освобождает. В языке нет сборщика мусора, поэтому если мы начинаем работать с динамической СД, мы за собой должны ее очистить 

Мы можем создать один объект какого-то типа

```cpp
int *px = new int(10);
```

мы выделили память на объект типа `int` и в конце надо будет очистить ее.

```cpp
delete px;
```

Такие объекты имеют динамическое размещение.

Можно например так завести массив

```cpp
int * px = new int[10];
```

`px` - указатель на первый элемент. Но чтобы освободить память надо будет

```cpp
delete [] px;
```

Конечно, форму оператора `new` , которая выделяет массив, можно передавать в значении известное только на этапе исполнения

```cpp
int main(int argc, char ** argv) {
    /* ... */
    int * px = new int[argc];
    /* ... */
}
```

Давайте попробуем реализовать динамический массив

```cpp
#define OUT(x) std::cout << x << " ";
#define LOOP(to_size) for (std::size_t i = 0; i < to_size; ++i)

class Rational
{
    int m_numerator = 0;
    int m_denominator = 1;
public:
    Rational() = default;
    Rational(const int n)
        : m_numerator(n) {}
    std::ostream & print(std::ostream & strm) const;
    friend std::ostream & operator << (std::ostream & strm, const Rational & r)
    {
        return r.print(strm);
    }
};

std::ostream & Rational::print(std::ostream & strm) const
{
    return strm << m_numerator << "/" << m_denominator;
}

class Vector
{
    Rational * m_data = nullptr; // нам поотребуется хранить указатель на первый эл-т
    std::size_t m_size = 0; // std::size_t беззнаковый целый тип
public:
    Vector() = default;

    Vector(const std::size_t size, const Rational & init = Rational{})
        :m_data(new Rational[size])
        ,m_size(size) {
            for (std::size_t i = 0; i < size; i++)
            {
                m_data[i] = init;
            }
    }

    Rational & operator [](const std::size_t i) { return  m_data[i]; }
    std::size_t size() const { return m_size; }
    const Rational & operator [](const std::size_t i) const { return m_data[i]; }

    void push_back(const Rational & r);

    void swap(Vector & other)
    {
        std::swap(m_data, other.m_data);
        std::swap(m_size, other.m_size);
    }

    virtual ~Vector();
};

void Vector::push_back(const Rational &r) {
    Vector tmp{m_size + 1};
    LOOP(m_size) {
        tmp[i] = m_data[i];
    }
    tmp[m_size] = r;
    swap(tmp);
}

Vector::~Vector()
{
    delete [] m_data;
}

int main() {
    Vector v(5, 3);
    v.push_back(11);
    for (std::size_t i = 0; i < v.size(); i++)
    {
        OUT(v[i]);
    }
}
```

Добавим еще метод удаления

```cpp
void Vector::pop_back() {
    if (m_size > 0) {
        Vector tmp{m_size - 1};
        LOOP(m_size - 1) {
            tmp[i] = m_data[i];
        }
        swap(tmp);
    }
}
```

реализуем еще print

```cpp
std::ostream  & print(std::ostream & strm) const {
        LOOP(m_size) {
            if (i != 0) {
                strm << " ";
            }
            strm << m_data[i];
        }
        return strm;
    }
```

надо будет для этого еще сделать

```cpp
friend std::ostream & operator << (std::ostream & strm, const Vector & v)
{
		return v.print(strm);
}
```

теперь проверим работу

```cpp
int main() {
    Vector v(5, 3);
    Vector vv = v;
    v.push_back(11);
    v.pop_back();
    v.pop_back();
    std::cout << "v: " << v << "\n" << "vv: " << vv << "\n";
}
```

Теперь наша программа неправильно работает)

```
v: 3/1 3/1 3/1 3/1
vv: 463215824/426 463208784/426 3/1 3/1 3/1

Process finished with exit code -1073740940 (0xC0000374)
```

Пошло не так то - что операция копирования генерируется автоматически. У нас получились два указателя, которые указывает на одну и ту же область памяти. Печать испортилась потому что, мы в методе `pop_back` на самом деле выделяем новую память, а старую удаляем, то есть  `vv` продолжает указывать на память, которую мы изначально выделили, которую мы теперь освободили, и теперь она непонятно куда указывает. 

Нам нужно реализовать свои операции копирования

```cpp
Vector(const Vector & other)
        :m_data(new Rational[other.m_size])
        ,m_size(other.m_size) {
        std::copy(other.m_data, other.m_data + m_size, m_data);
    }
```

если мы определили конструктор копирования, то нам нужно определить и оператор копирующего присваивания 

```cpp
Vector & operator = (const Vector & other)
    {
        auto tmp = other;
        swap(tmp);
        return *this;
    }
```

Можно и переписать `pop_back` . Мы сейчас память перевыделяем, когда можно было просто уменьшить логический размер вектора.

Что если мы хотим вернуть вектор,  как результат работы какой-то функции.

Мы можем написать как-то так

```cpp
Vector f()
{
		Vector v;
		/* ... */
		return v;
}
```

Мы все элементы копируем, а потом удаляем, это не очень эффективно. Можем передавать вектор по ссылке 

```cpp
Vector f(Vector & v)
{
		/* v = ... */
}
```

вызывать ее будем как

```cpp
Vector v;
f(v)
```

вернула нам вектор, но копирование не случилось. Хорошо, но не очень удобно. Если нам передадут что-то не то, то наши ожидания немного поламались.

Может так?

```cpp
void g(Vector & v)
{
		Vector vv;
		v.swap(v);
}
```

но мы потом ошиблись и продолжили работу с v

```cpp
g(vv)
std::cout << vv;
```

Человек может  заметить такую ошибку, но для компилятора это не очевидно. Нам нужен механизм, чтобы обезопасить такие ситуации. Чтобы четко различить отдаем ли мы объект  с концами, либо мы отдаем объект просто попользаваться. 

Для этого в 11м стандарте были введены `rvalue`ссылки 

```cpp
void g(Vector && v) 
{
		Vector vv;
		v.swap(v);
}
```

но теперь вызов функции меняется. Надо будет заиспользовать библиотечную функцию `std::move` 

```cpp
g(std::move(vv));
```

но теперь, конечно, мы не можем использовать объект `vv`.

Но не для всех типов `move` более эффективна чем копирование. Работа с памятью - классический пример, где мы можем получить эту эффективность.

Конструктор перемещения и перемещающий оператор присваивания.

```cpp
Vector(Vector && other)
    :m_data(other.m_data)
    ,m_size(other.m_size)
{
    other.m_data = nullptr;
    other.m_size = 0;
}
```

Разница с копирующим конструктором очевидана. Мы не выделяем память, мы не копируем ничего, кроме указателя. Надо только не забыть в доноре указатель обнулить. То есть, по сути, мы передаем владение какой-то области памяти. 

Заметим, что для `Rational` копирование ничем бы особо не отличалось, и там нету смысла явно объявлять эту операцию, компилятор ее делает автоматически.

Присваивание будет выглядеть также

```cpp
Vector & operator = (Vector && other)
{
		swap(other);
		return *this;
}
```