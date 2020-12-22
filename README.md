_Sorry, I don't know enough English to write this documentation in it. I will
be glad to help._

# format

format - это пакет для форматирования строк на Дарте. Сейчас в нём только одна
функция, собственно, format().

## Содержание
- [format()](#stringformat)
    - [Пример использования](#пример-использования)

## String.format()

Функция-расширение класса [String](https://api.dart.dev/stable/dart-core/String-class.html),
аналогичная методу [format](https://docs.python.org/3/library/string.html#format-string-syntax)
в Python, функции [std::format](https://en.cppreference.com/w/cpp/utility/format/format)
из С++20, которые в свою очередь стали развитием популярной функции [sprintf](https://en.cppreference.com/w/c/io/fprintf)
из C. Суть её в том, чтобы вместо шаблонов, заключённых в фигурные скобки `{}`,
подставить значения переданных аргументов, отформатировав их требуемым образом.

```dart
String result = '{...}'.format(...);
```

### Описание шаблона

```
template            ::=  '{' [argId] [':' formatSpec] '}'
argId               ::=  index | identifier | doubleQuotedString | singleQuotedString
index               ::=  digit+
identifier          ::=  idStart idContinue*
idStart             ::=  '_' | letter
idContinue          ::=  '_' | letter | digit
letter              ::=  <любая буква любого языка> (\p{Letter})
doubleQuotedString  ::=  '"' <любые символы, с заменой " на ""> "'"
singleQuotedString  ::=  "'" <любые символы, с заменой ' на ''> '"'
arg_name            ::=  [identifier | digit+]
attribute_name      ::=  identifier
formatSpec          ::=  <в следующем разделе>
```

### Синтаксис строки форматирования

```
formatSpec      ::=  [[fill]align][sign][#][0][width][grouping_option][.precision][type]
fill            ::=  <любые символы>
align           ::=  '<' | '>' | '^'
sign            ::=  '+' | '-' | ' '
width           ::=  digit+ | '{' argId '}'
groupingOption  ::=  '_' | ','
precision       ::=  digit+ | '{' argId '}'
type            ::=  'b' | 'c' | 'd' | 'e' | 'E' | 'f' | 'F' | 'g' | 'G' | 'n' | 'o' | 's' | 'x' | 'X'
```

#### `fill` и `align`

Если выравнивание `align` задано, то строка заполнения `fill` может быть любым
сочетанием символов, в то время как в Python допустим только один символ. Это
сделано для того, чтобы можно было использовать символы, состоящие из нескольких
Unicode-знаков (символы с диакретическими знаками, суррогатные пары и т.д.),
и при этом не заниматься лишним анализом.

Python накладывает только одно ограничение на символ заполнения - нельзя
использовать фигурные скобки `{}`, т.к. они используется для вставки значений 
внутрь шаблона. В Dart в такой функции необходимости нет, т.к. у него есть 
встроенная в язык прекрасная [string interpolation][https://dart.dev/guides/language/language-tour#strings].

Возможные значения `align`:

| Значение | Описание
| :---:    | ---
| '<'      | Выравнивание влево (действует по умолчанию для строк и символов).
| '>'      | Выравнивание вправо (действует по умолчанию для чисел).
| '^'      | Выравнивание по центру.

Разумеется, `align` имеет значение только тогда, когда задана минимальная ширина
поля `width`, и она больше, чем реальная ширина поля. В этом случае результат 
дополняется до заданной с соответствующей стороны.

Python включает ещё значение '=', посредством которого можно вставить любой 
символ вместо ведущих нулей между знаком числа ('+' или '-') и первыми 
значащими цифрами ('+    123', '-···42'). '0' перед шириной действует по тому же
принципу. Я не стал реализовывать такую возможность. Во-первых, потому что я не 
могу понять причин, зачем это нужно. Если была задача заменить ведущие нули
любым символом, то это не совсем так, потому что опция расстановки разделителей
тысяч (',') не влючается, если указан символ, отличный от '0'. Т.е. Python
в данной функции отличает нули от прочих символов. Во-вторых, из-за этой
особенности появляются все эти странные '00000nan', '-0000inf'. Хотя и не
понятно, почему именно в этих случаях исключение для нулей сделано не было.

#### `sign`

С помощью этой функции можно указать, как обращаться со знаком числа.

| Значение | Описание
| :---:    | ---
| '-'      | Знак ставится только у отрицательных чисел (это значение по-умолчанию).
| '+'      | У положительных чисел тоже ставится знак (ноль тоже идёт с плюсом).
| ' '      | У положительных чисел вместо '+' ставится символ пробела. Это может пригодиться для выравнивания положительных и отрицательных чисел между собой.


### Пример использования

```dart
import 'package:format/format.dart';
import 'package:intl/intl.dart';

void main() {
  print('hello {}'.format(['world'])); // hello world
  print('{} {}'.format(['hello', 'world'])); // hello world
  print('{1} {0}'.format(['hello', 'world'])); // world hello
  print('{} {} {0} {}'.format(['hello', 'world'])); // hello world hello world
  print('{a} {b}'.format([], {'a': 'hello', 'b': 'world'})); // hello world

  print('{a} {"a"} {"+"} {"hello world"}'.format([], {'a': 1, '+': 2, 'hello world': 3})); // 1 1 2 3
  print("{a} {'a'} {'+'} {'hello world'}".format([], {'a': 1, '+': 2, 'hello world': 3})); // 1 1 2 3

  const n = 123.4567;
  print('{:.2f}'.format([n])); // 123.46

  print('{:10.2f}'.format([n])); // '    123.46'
  print('{:<10.2f}'.format([n])); // '123.46    '
  print('{:^10.2f}'.format([n])); // '  123.46  '
  print('{:>10.2f}'.format([n])); // '    123.46'

  print('{:*<10.2f}'.format([n])); // 123.46****
  print('{:*^10.2f}'.format([n])); // **123.46**
  print('{:*>10.2f}'.format([n])); // ****123.46

  print('{:010.2f}'.format([n])); // 0000123.46
  print('{:012,.2f}'.format([n])); // 0,000,123.46
  print('{:012_.2f}'.format([n])); // 0_000_123.46

  print('{:0{},.{}f}'.format([n, 12, 2])); // 0,000,123.46
  print('{value:0{width},.{precision}f}'.format([], {'value': n, 'width': 12, 'precision': 2})); // 0,000,123.46

  const n1 = 123456.789;
  const n2 = 1234567.89;
  print('{:g}'.format([n1])); // 123457
  print('{:g}'.format([n2])); // 1.23457e+6
  print('{:.9g}'.format([n1])); // 123456.789
  print('{:.9g}'.format([n2])); // 1234567.89
  print('{:.5g}'.format([n1])); // 1.2346e+5
  print('{:.5g}'.format([n2])); // 1.2346e+6

  print('{:g}'.format([double.nan])); // nan
  print('{:g}'.format([double.infinity])); // inf
  print('{:g}'.format([double.negativeInfinity])); // -inf

  const i = 12345678;
  print('{:b}'.format([i])); // 101111000110000101001110
  print('{:d}'.format([i])); // 12345678
  print('{:x}'.format([i])); // bc614e
  print('{:X}'.format([i])); // BC614E
  print('{:#x}'.format([i])); // 0xbc614e
  print('{:#X}'.format([i])); // 0xBC614E

  print('{:_b}'.format([i])); // 1011_1100_0110_0001_0100_1110
  print('{:,d}'.format([i])); // 12,345,678
  print('{:_d}'.format([i])); // 12_345_678
  print('{:_x}'.format([i])); // bc_614e
  print('{:_X}'.format([i])); // BC_614E
  print('{:#_x}'.format([i])); // 0xbc_614e
  print('{:#_X}'.format([i])); // 0xBC_614E

  print('{:c}+{:c}+{:c}+{:c}={:c}'.format([
    0x1F468, // 👨
    0x1F469, // 👩
    0x1F466, // 👦
    0x1F467, // 👧
    [0x1F468, 0x200D, 0x1F469, 0x200D, 0x1F466, 0x200D, 0x1F467], // 👨‍👩‍👦‍👧
  ]));

  const m = 12345678.9;
  Intl.defaultLocale = 'ru_RU';
  print('{:n}'.format([m])); // 1,23457E7
  print('{:.9n}'.format([m])); // 12345678,9
  print('{:012,.9n}'.format([m])); // 12 345 678,9
  print('{:n}'.format([double.nan])); // не число
  print('{:n}'.format([double.infinity])); // ∞
  print('{:n}'.format([double.negativeInfinity])); // -∞

  Intl.defaultLocale = 'de_DE';
  print('{:n}'.format([m])); // 1,23457E7
  print('{:.9n}'.format([m])); // 12345678,9
  print('{:012,.9n}'.format([m])); // 12.345.678,9

  Intl.defaultLocale = 'en_IN';
  print('{:n}'.format([m])); // 1.23457E7
  print('{:.9n}'.format([m])); // 12345678.9
  print('{:012,.9n}'.format([m])); // 1,23,45,678.9

  Intl.defaultLocale = 'bn';
  print('{:n}'.format([m])); // ১.২৩৪৫৭E৭
  print('{:.9n}'.format([m])); // ১২৩৪৫৬৭৮.৯
  print('{:012,.9n}'.format([m])); // ১,২৩,৪৫,৬৭৮.৯

  Intl.defaultLocale = 'ar_EG';
  print('{:n}'.format([m])); // ١٫٢٣٤٥٧اس٧
  print('{:.9n}'.format([m])); // ١٢٣٤٥٦٧٨٫٩
  print('{:012,.9n}'.format([m])); // ١٢٬٣٤٥٬٦٧٨٫٩
  print('{:n}'.format([double.nan])); // ليس رقم
}
```
