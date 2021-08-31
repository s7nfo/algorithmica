---
title: Корневые структуры
authors:
- Иван Сафонов
---

Рассмотрим следующую учебную задачу: Дан массив $a_1,\\ a_2,\\
\\ldots,\\ a_n$. Надо обрабатывать два типа запросов:

  - Найти сумму на отрезке $\[l;\\ r\]$.
  - Увеличить значение элемента $a_{pos}\\ +=\\ x$

### Решение

Разобьем массив на блоки размера $K = O(\\sqrt n)$. Кроме обычной
последовательности, будем хранить сумму чисел в блоке. При
запросе обновления можно за $O(1)$ обновить элемент
последовательности и пересчитать соответствующий блок. Для
get-запроса просмотрим все блоки, которые находятся внутри отрезка
запроса, и прибавим к ответу посчитанную сумму. А также отдельно
рассмотрим все элементы, которые не попали ни в один блок. Количество
нужных нам блоков — это $O(\\frac{n}{k}) = O(\\sqrt n)$, число
единичных элементов также $O(k) = O(\\sqrt n)$.

``` c++ numberLines
int a[MAXN];
int b[MAXN];
int k = 400; // предпочтительно писать константу

void build() {
    for (int i = 0; i < n; i++) {
        b[i / k] += a[i];
    }
}

int sum(int l, int r) {
    int res = 0;
    while (l <= r) {
        if (l % k == 0 && l + k - 1 <= r) {
            res += b[l / k];
            l += k;
        }
        else {
            res += a[l];
            l += 1;
        }
    }
    return res;
}

void upd(int pos, int x) {
    a[pos] += x;
    b[pos / k] += x;
}
```

#### Изменение на отрезке

В корневой можно пользоваться техникой [отложенных
операций](отложенные_операции "wikilink"). Это
выглядит вот так: мы запомним $push$-величину, которую будем
проталкивать перед обращению к $\\textit{одиночным}$ элементам
$a_i$

``` c++ numberLines
void push(int block) {
    if (add[block] == 0) {
        return;
    }
    for (int i = block * k; i < min(n, (block + 1) * k); i++) {
        a[i] += add[block];
    }
    add[block] = 0;
}

void upd(int l, int r, int x) {
    while (l <= r) {
        if (l % k == 0 && l + k - 1 <= r) {
            b[l / k] += k * x;
            add[l / k] += x;
            l += k;
        }
        else {
            a[l] += x;
            l++;
        }
    }
}
```

#### Отсортированные последовательности

В блоках корневой можно хранить не только значения функций для
подотрезка, а еще и его отсортированную версию. Это бывает
полезно при ответе на запросы вида "число меньших $x$ на отрезке" и
используется в техниках [split-rebuild](split-rebuild "wikilink") и
[split-merge](split-merge "wikilink").

[Категория:Конспект](Категория:Конспект "wikilink") [Категория:Структуры
данных для запросов на
отрезках](Категория:Структуры_данных_для_запросов_на_отрезках "wikilink")

---

Простое разделение на блоки позволяет создать достаточно мощную
структуру, которая позволяет делать операции вставки, удаления
и подсчета некоторых функций на отрезках за корень. Рассмотрим задачу
подробнее.

## Постановка задачи

Пусть дан массив из $N$ элементов. К нему поступает $M$ запросов трех
видов:

  - Вставить элемент $x$ на позицию $i$, то есть слева от него окажется
    ровно $i$ элмементов;
  - Удалить элемент с позиции $i$;
  - Найти минимум на полуинтервале $\[l, r)$.

Будем считать, что все индексы начинаются с нуля.

## Решение с помощью корневой декомпозиции

Основная идея решения: разделим массив <em>примерно</em> на $\\sqrt{N}$
частей одинакового размера. Будем поддерживать глобальный счетчик
`OPERATIONS_COUNTER`, равный числу операций изменения, произведенных над
структурой.

### Вставка

При вставке будем явно вставлять элемент в нужный блок. Если вставка
происходит на границе блоков, то договоримся вставлять элемент в
единственный существующий блок, если вставка производится в самый
конец или в самое начало. Иначе вставляем в блок, найденный функцией
`findBlock`. Можно сделать иначе при вставке на концах - вставлять в
новый блок, создавая его. Этот случай будет лучше работать, если у
нас происходит много вставок подряд, а предыдущий способ - когда
больше надо отвечать на запросы. Можно также принимать решение о
вставке тем или иным способом случайно. Это решение, наверное, будет
наиболее универсальным и применимым.

``` c++ numberLines
void insert(int pos, int elem) {
    OPERATIONS_COUNTER++;
    if (blocks.empty()) {
        blocks.resize(1);
        blocks[0].push_back(elem);
        return;
    }
    pair<int, int> posBlock = findPosition(pos);
    if (posBlock.first == blocks.size()) {
        blocks.back().push_back(elem);
        return;
    }
    blocks[posBlocks.first].insertAtPosition(posBlock.second, elem);
```

### Удаление

При удалении будем явно удалять элемент из блока за размер блока. Если
блок оказался пустым, то ничего с ним не будем делать пока что.

### Функция на полуинтервале

Запрос о вычислении функции обрабатываем, как в обычной простой корневой
декомпозиции.

### Перестраивание

Чтобы на допускать создания слишком большого числа маленьких блоков или
разрастания отдельных блоков, раз в $\\sqrt{N}$ операций будем заново
полностью перестраивать структуру. Размер блока при этом тоже будет
меняться. Также будем всегда поддерживать размер всей структуры. Мало
ли, может пригодиться.

Предположим, что массив блоков хранится как глобальный вектор `blocks`.

``` C++ numberLines
void rebuild() {
    int newBlockSize = sqrt(size) + 1;
    vector<int> all;
    for (auto& block : blocks) {
        for (auto elem : block) {
            all.push_back(elem);
        }
    }
    blocks.clear();
    int newBlockNumber = (totalSize + newBlockSize - 1) / newBlockSize;
    blocks.resize(totalSize);
    for (int i = 0; i < totalSize; i++) {
        blocks[i / newBLockSize].push_back(all[i]);
        updateFunction(i);
    }
}
```

### Как искать нужный блок

Так как теперь мы не можем гарантировать, что блок имеет фиксированный
размер в каждый момент времени, то находить нужное место будем просто
проходом по массиву блоков. Функция возвращает номер блока и позицию
именно в нем.

``` C++ numberLines
pair<int, int> findBlock(int pos) {
    if (blocks.empty() || pos == 0) {
        return {0, 0};
    }
    int byLeft = 0;
    int i = 0;
    while (i < blocks.size() && byLeft < pos) {
        byLeft += blocks[i].size();
        i++;
    }
    i--;
    byLeft -= blocks[i].size();
    return {i, pos - byLeft};
}
```

---

Пререквизиты: [Split-rebuild](Split-rebuild "wikilink")

## Split-merge

### Идея

В рамках структуры данных [split-rebuild](split-rebuild "wikilink") мы
регулярно перестраивали структуру данных. Оказывается, бывают
ситуации, когда перестраивать структуру данных не так выгодно,
как поддерживать ее в сбалансированном состоянии.

### Задача

Дан массив $a_n$, с которым надо выполнять следующие операции:

  - Вставка и удаление из произвольной позиции
  - Ответ на запрос "количество $x \\ge a$" на отрезке $\[l:r\]$
  - Массовые операции (например, переворот отрезка)

Заметим, что <st>это какая-то жесть</st> эту задачу мы уже могли решить
с помощью [split-rebuild](split-rebuild "wikilink"). Для этого нам надо
было бы хранить для каждого блока его отсортированную версию. Если для
каждого отсортированного элемента хранить его исходный индекс, то
можно будет делать \`split\`за линейное время. Таким образом, у нас
будет асимптотика $O(q \\sqrt{n} \\log n)$, ведь теперь для \`rebuild\`
и \`lower\\_bound\` нам понадобится сортировка.

### Склеиваем блоки

Теперь заметим, что мы можем склеить два соседних маленьких блока за
$O(K)$, где $K$ --- размер блока. Для этого нам понадобится стандартный
[merge](merge "wikilink"). Тогда будем склеивать блоки, если существует
пара соседних блоков, каждый из которых меньше, чем $\\frac{K}{2}$. А
резать блок будем, если его размер больше $2K$. Тогда блоков всегда
будет $O(\\frac{n}{K})$, а размер блоков будет $O(K)$.

### В чем выгода?

Теперь нам не надо будет делать сортировку. Для \`get\`-запросов
сложность осталась равной $O(q K \\log n)$, но теперь мы не
перестраиваем блоки, а склеиваем их за $O(\\frac{qn}{k})$. Тогда
при выборе $K = \\sqrt{\\log n}$ получаем сложность $O(q \\sqrt{n
\\log n})$.

[Категория:Конспекты](Категория:Конспекты "wikilink")