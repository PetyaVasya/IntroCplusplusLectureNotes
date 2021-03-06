# Lecture 5b: Продолжение

Возвращаемся к конструкторам.

```cpp
Rational() = default;
```

```cpp
Rational(const int a, const int b)
```

```cpp
Rational(const double x)
    : numerator(x)
```

На самом деле, 3й конструктор (конструтор при одном аргументе), можно было и не писать, могли бы отделаться, написав

```cpp
Rational(const int a, const int b = 1)
```

Но, что интересно, про конструкторы от одной переменной, это то, что они начинают участвовать в неявном преобразовании типов

```cpp
auto r1 = Rational::read(std::cin);
if (r1) {
    r1->add({11})
    /* ... */
```

Но теперь, вся процедура неявоного приведения типов - усложняется. Теперь она фактически делится на 3 этапа…

1. Стандартные неявные преобразования. 

Сюда может входить: числовое расширение, числовое приведение… 

1. Пользавтельское преобразлвание
2. Вновь стандартное преобразование

Но 3 этап может только случиться, если был 2й этап.

```cpp
r1->add(11)
```

Что случится, если мы напишем 

```cpp
r1->add(11L);
```

Тип `long` можно привести как и к `int` так и к `double` . Компилятор видит 2 подходящих варианта, оба требуют первый шаг (стандартное преобразование), в обоих случаях это преобразование более менее одинаково, в одном случае приводим от более широкого к менее широкому, во втором от целого к дробному. В итоге выходит ошибка

Но если бы мы сделали  так

```cpp
short x = 11;
r1->add(x);
```

Ошибки - не будет. Происходит расширене типа до `int` (все целочисленные типы расширяются до `int` кроме `float` , который расширяется до `double`). Компилятор считает, что расширение немного лучше, чем преобразование, поэтому у компилятора не вопросов, и все работает.

С пользавательскими преобразованиями количество вариантов увеличивается и, возможно, не все являются желательными. Как этого можно избежать? 

Предположим, мы считаем, что наше преобразование из `double` в `Rational` требующим большего внимания. Можем ли мы обязать его это преобразование делать явно? Просто убрать конструктор нельзя. Ему придется процедуру из `double` к `Rational` делать через `int` . 

Что если мы хотим определить эту процедуру, но обязять пользавателя программиста ее явно запрашивать. Мы можем это сделать

```cpp
explicit Rational(const double x)
    : numerator(x)
```

теперь у нас компилируется 

```cpp
r1->add(11L);
```

В этом случае, компилятор всегда выбирает первый конструктор. Что будет, если мы напишем число с плавающей точкой

```cpp
r1->add(0.1);
```

Все скомпилировалось, но вызывается первый конструктор `double → int` . Это не очень хорошо, поэтому компилятор выдает предупреждение. 

Но, если мы явно указываем имя типа, то для компилятора это является флагом того, что мы запрашиваем явное преобразование.

```cpp
r1->add(Rational{0.1});
```

Теперь начинает работать конструктор, который помечен с `explicit` .

Но к сожалению нас `explicit` не избавил от всех проблем, то есть, если мы не будем указывать явно, то у нас вызовется не тот конструктор. 

Спецификатор `explicit` делает ровно одну вещь, он убирает сущность к которой он приписан из цепочки неявных преобразований. 

Что, если мы хотим сделать наоборот, то есть из тип `Rational` получать тип `double` .

```cpp
double to_double() const {
    return static_cast<double>(numerator) / denominator;
}
```

Везде, где нам хочется этого преобразования нам надо вызывать этот метод. Иногда нам может захотеться неявного преобразования. Как этого достичь?

Язык позволяет нам  определить оператор приведения типов. 

Этот оператор имеет такой вид

```cpp
operator double () const {
    return to_double();
}
```

Обычно у функций и операторов есть тип возвращаемого значения, можно было предположить, что сначала должен был бы писаться `double`, но создатели языка увидели избыточность, мы дважды пишем слово `double` . Теперь происходит преобразование из пользавательского в базовый тип. 

Эти операторы еще один случай, когда мы можем использовать спецификатор `explicit` . Теперь, чтобы пользоваться этим нужно будет явно писать `static_cast`   

### Копирование

На самом деле копирование можно разделить на 2 случая

```cpp
auto tmp = *this;
```

Мы инициализируем новый объект нашего класса путем копирования значения из другого объекта этого класса. Здесь можно представить, что инициализация ссылочного поля тоже очевидно. У нас уже есть объект с таким ссылочным полем, оно уже было когда-то проинициализировано, значит мы знаем, что оно указывает на корректный объект. 

```cpp
A(const A & other)
    : n(other.n)
{

}
```

Но теперь, что делать в такой ситуации? 

```cpp
A a1, a2;
a1 = a2;
```

Мы не можем переназачить ссылку, поэтому тут нет какого-то разумного поведения по умолчанию. Компилятор это за нас делать не будет, по этой причине.

Определим операцию присваивания для `Rational` . Это тоже будет являться перегруженным оператором.

```cpp
void operator = (const Rational & other)
{
    numerator = other.numerator;
    denominator = other.denominator;
} 
```

Тут все достаточно тривиально, поэтому у нас и работала раньше эта операция, компилятор ее сам сгенерил. Это копирующий оператор присваивания.

В общем случае, операция копирования имеет два случая.

1. Инициализация копирования 
2. Присваивание копирование 

Для инициализации копирования служит конструктор, который называется конструктор копирования, а для присваиавания копирования служит оператор.

Сейчас у нашего оператора есть одна проблема, в общем случае, мы можем писать вот так

```cpp
a = b = c = d;
```

Это работает потому что встроенный оператор присваивания возвращает какое-то значение, из которого можно прочитать то значение, которое было присвоено, и засчет правил ассоциативности у нас получится

```cpp
a = (b = (c = d));
```

То есть из оператора присваиавания можно делать сложные выражения, а из нашего пока нельзя, потому что он ничего не возвращает 

```cpp
Rational & operator = (const Rational & other)
{
    numerator = other.numerator;
    denominator = other.denominator;
    return *this;
}
```

Это будет корректно, потому что в левый объект записываем значение из правого, а потом возвращаем ссылку на левый объект. 

### Методы класса, которые могут быть сгенерены автоматически

1. Конструктор по умолчанию
2. Конструктор копирования
3. Копирующий оператор присваивания

Сгенерены автоматически значит, что мы могли бы написать вот так, или просто упустить

```cpp
Rational & operator = (const Rational & other) = default;
```

Для конструктора копирования  и копирующего оператора присваивания главное условие - это наличие разумной операции копирования по умолчианию (она может быть определена) 

Как понять, что это конструктор копирования 

Конструктор копирования должен быть не шаблонным и должен уметь вызываться с одним аргументом, который должен являться `lvalue` ссылкой на объект того же типа. Разумно определять конструктор копирования с константной ссылкой, потому что обычно мы  не хотим модифицировать того из чего мы копируем. 

Что если мы сделаем такой конструктор

```cpp
A(A & other)
{
		
}
```

теперь у нас `rvalue` ссылка. Этот конструктор является конструктором перемещения. 

Теперь оператор присваивания будет иметь такой вид

```cpp
A & operator = (A && other) 
{
    return *this;
}
```

будет возвращать `lvalue` ссылку. Такое дело тоже может быть сгенерено автоматически

Также есть случай, когда мы можем пометить специальный метод как удаленный 

```cpp
Rational() = delete;
```

Это значит, что мы запрещаем компилятору неявно автоматически генерить этот метод. Например, мы хотим запретить операцию копирования для нашего класса, мы это можем сделать вот так

```cpp
Rational(const Rational &) = delete;
Rational & operator = (const Rational &) = delete;
```

Здесь операция копирования для нашего класса запрещена.

### Cравнение

```cpp
auto operator == (const Rational & x) const
{
    return numerator == x.numerator && denominator == x.denominator;
}
```

В 20м стандарте языка в языке ввели новый опреатор `<=>` в связи с тем, что сравнивать объекты не так уж и просто. Этот опрератор носит название космический корабль (spaceship).

Мы его можем определить как

```cpp
auto operator <=> (const Rational & x) const
{
    return numerator <=> x.numerator;
}
```

Теперь для пользавательского типа достаточно определить эти два этих оператора. Они не являются взаимозаменяемые, при этом компилятор их умеет генерить автоматически. 

Что теперь происходит с остальными операторами сравнения ≠ , ≤ итд. Ввели в язык еще одно усложнение и сказали, что компилятор не будет генерировать эти операторы, но в тех местах кода, в которых используются такие операторы сравения.

Если будет написано

```cpp
r1 != r2;
```

то он представит, что будет написано 

```cpp
!(r1 == r2);
```

Если будет написано

```cpp
r1 < r2;
```

то он представит в виде

```cpp
(r1 <=> r2) < 0;
```

Что важно, это что методов новых для класса он генерировать не будет. Он переписывает наш код, делая вид, что мы написали другой код. Это не очень круто, но ничего лучше не придумали.

Стоит запомнить то, что теперь достаточно только определить оператор равенства.
