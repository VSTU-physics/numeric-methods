---
layout: post
title:  "Численное интегрирование: интерполяция многочленом"
date:   2014-09-23 21:12:19
categories: интегрирование
---

Рассмотрим задачу о нахождении интеграла

$$
    I = \int\limits_a^b f(x) dx.
$$

Приближённо это значение можно определить, приблизив подынтегральную функцию
многочленом:

$$
    f(x) \approx P_n(x) = a_0 + a_1 x + \cdots + a_n x^n,
$$

при этом получается следующее приближение для интеграла:

$$
    I = \int\limits_a^b f(x)\,dx \approx
    \int\limits_a^b\sum_{k=0}^n a_k x^k\,dx =
    \sum_{k=0}^n \frac{a_k(b^{k+1} - a^{k+1})}{k+1}.
$$

Для построения многочлена $$ P_n(x) $$ нужно определить $$ n+1 $$ коэффициентов
$$ a_i $$. Их можно определить, выбрав на $$ [a, b] $$ точки
$$ \{ x_0, x_1, \ldots, x_n \} $$ и составив систему уравнений

$$
  P_n(x_i) = \sum_{k=0}^n a_k x_i^k = f(x_i) \quad i\in\{0,\ldots,n\},
$$

которую можно переписать в матричном виде:

$$
    \begin{pmatrix}
        1 & x_0 & \cdots & x_0^n \\
        1 & x_1 & \cdots & x_1^n \\
        \vdots & \vdots & \ddots & \vdots \\
        1 & x_n & \cdots & x_n^n \\
    \end{pmatrix}
    \begin{pmatrix}
    a_0 \\ a_1 \\ \vdots \\ a_n
    \end{pmatrix}
    =
    \begin{pmatrix}
    f(x_0) \\ f(x_1) \\ \vdots \\ f(x_n)
    \end{pmatrix}
$$

Эта _система линейных алгебраических уравнений (СЛАУ)_ содержит $$ n + 1 $$
уравнение и столько же неизвестных. Если все точки $$ \{x_i\} $$ различны, то
определитель матрицы этой системы отличен от нуля и она имеет единственное
решение. Его можно найти численно, используя, например,
[LUP-разложение]({{ "/СЛАУ/2014/09/28/LUP/" | prepend: site.baseurl }}).

По традиции, код для [python 2.7](http://python.org):
{% highlight python %}
# coding: utf8

def polynomial(f, a, b, n):
    '''
        Интегрирование f(x) на [a, b] при помощи
        интерполяционного многочлена порядка n
    '''

    # формируем массив точек для интерполяции
    # для простоты рассмотрим равномерное распределение
    xs = [a + (b - a) * float(i) / n for i in range(n + 1)]

    # формируем матрицу системы и столбец свободных членов
    A = [[x ** j for j in range(n + 1)] for x in xs]
    B = [f(x) for x in xs]

    # определяем коэффициенты многочлена
    # пример такой функции описан в статье про LUP-разложение
    ps = solve(A, B)

    # считаем ответ
    return sum(ps[i] * (b ** (i + 1) - a ** (i + 1)) / (i + 1)
               for i in range(n + 1))
{% endhighlight %}

Вдумчивый читатель может заметить, что если края отрезка интегрирования имеют величину порядка
10, то уже при интерполировании кубическим четырёхчленом в матрице появляются элементы порядка
1000. В этом случае, с ростом $$ n $$ элементы матрицы будут расти экспоненциально,
что может привести к существенным неточностям.

Для решения этой проблемы можно воспользоваться преобразованием координат. Пусть

$$
  g(t) = \frac{b - a}{2} t + \frac{b + a}{2},\ g(-1) = a,\ g(1) = b,
$$

тогда

$$
    \int\limits_a^b f(x) dx = \frac{b-a}{2}\int\limits_{-1}^1 f(g(t)) dt =
    \frac{b-a}{2}\int\limits_{-1}^1 \tilde{f}(t) dt,\ \tilde{f} = f \circ g.
$$

Так как узлы по модулю не превосходят 1, то при их возведении в степень они
не будут возрастать и точность будет выше.