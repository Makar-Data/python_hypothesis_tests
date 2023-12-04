# Тесты гипотез в Python
Набор кода Python по статистическим тестам с комментариями.

Flowchart by [Antoine Soetewey](https://statsandr.com/blog/what-statistical-test-should-i-do/) для определения необходимого теста
![alt text](https://github.com/Makar-Data/python_hypothesis_tests/blob/main/Antoine_Soetewey_flowchart.png?raw=true)

Группы тестов
Конкретные тесты с требованиями (и гиперссылкой на них в тексте?)

### Стьюдент-Т-тест для одной выборки
- Предполагает, что ГС - нормальная
- Лучше не использовать на небольшие выборки
```Python
results = stats.ttest_1samp(a = minnesota_ages, popmean = population_ages.mean())  
std_bounds = stats.t.ppf(q=0.025, df=49) 

# q - отрезки и в начале, и в конце. Для доверительного интервала в 95% надо 2,5% и там, и там, поэтому ставим 0.025, так как нам нужно узнать начало std

# Результат даст #tstatistic и  #pvalue 

# tstatistic - отличие std выборки от std генеральной совокупности
# если выходит за рамки std_bounds, то значимо. В особых случаях людям нужна помощь

# pvalue - вероятность того, что наблюдение случайно (H0)
# если перешла низкий порог стат значимости, то значимость есть - мы пускаем её обездоленную домой
```

### Уилкоксон-тест для одной выборки 
```Python
from scipy.stats import wilcoxon
from scipy.stats import norm

wilcoxon(sample-np.median(general), zero_method="wilcox", correction=False)
# помимо pvalue, выдаст сумму рангов
# wilcox - сначала убирает нули перед назначением рангов значениям
# pratt - включит нули, но уберёт их ранг
# zsplit - включит нули и разделит ранги на положительные и отрицательные
z_value = norm.ppf(pvalue/2) # Нужно, если выборка слишком большая для критических значений Z-таблицы. Делим на 2 для двустороннего теста. Если > 1.96 для нормального распределения, то не принимаем Г0. Нужно, если выборка слишком большая для таблицы критических значений
```

### Стьюдент-Т-тест для двух связанных выборок
- Предполагает, что ГС - нормальная
- Лучше не использовать на небольшие выборки
```Python
results = stats.ttest_rel(a=before, b=after, axis=0, nan_policy="?")
std_bounds = stats.t.ppf(q=0.025, df=a+b-1)
```

### Уилкоксон-тест для связных выборок
```Python
scipy.stats.wilcoxon(x=before, y=after, axis=0, nan_policy="?", correction=False)
```

### Стьюдент-Т-тест для двух независимых выборок
- Предполагает, что ГС - нормальная
- Лучше не использовать на небольшие выборки
- Нужно понимать, есть ли равенство дисперсий

Тест Левена на равенство дисперсий для двух выборок
```Python  
levene = stats.levene(sample1, sample2, center="mean")

var_sig = []
if levene[1] < 0.05:  
    var_sig.append(False)  
else:  
    var_sig.append(True)

results = stats.ttest_ind(a = sample1, b = sample2, equal_var=var_sig)
std_bounds = stats.t.ppf(q=0.025, df=a+b-1)

# equal_var - есть ли гомогенность дисперсии двух выборок
# True - Стьюдента, False - Уэлча
# df для двух выборок = размер выборки один + размер выборки два - 1. Но вообще его показывает в результатах
```
# Fligner-Killeen_test самый популярный
# Brown_Forsythe_test набирает популярность

### Манна-Уитни-U-тест для двух независимых выборок
- Нужна одинаковое распределение у выборок
- Выборки меньше - получше
Сложнее интерпретировать
```Python
results = stats.mannwhitneyu(sample1, sample2)
  
u = results[0]  
mean = (len(sample1)*len(sample2))/2  
std = np.sqrt((len(sample1)*len(sample2)*(len(sample1)+len(sample2)+1))/12)  
z = (u-mean)/std

# для выборки < 20 критическое значение рассчитывается по таблице U
```

### Хи-квадрат-тест соответствия распределений категоричных значений
```Python
observed = sample_crosstab
general_ratios = general_crosstab/len(general)
expected = general_ratios * len(sample)

crit = stats.chi2.ppf(q=0.95, df=4)
# chi - односторонний, поэтому 0.95
# df - количество категорий минус 1

stats.chisquare(f_obs=observed, f_exp=expected) # expected - распределение, которые мы должны ожидать от генеральной совокупности, если выборка репрезентативна
# если chi_statistic > chi_critical, то есть статистическая значимость
```

### Хи-квадрат-тест независимости двух категорий
```Python
observed = sample_crosstab # без тоталов!

crit = stats.chi2.ppf(q=0.95, df=8) # измерения таблицы expected -1 на каждом измерении, помноженные друг на друга (таблица 3/5 = 8)

stats.chi2_contingency(observed=observed)
```

#№№ ANOVA - тест для более двух совокупностей
- Выборки независимы
- Есть гомогенность дисперсий ( #Alexander-Govern )
- Генеральные совокупности имеют нормальное распределение ( #Kruskal-Wallis #H-тест )
```Python
df = pd.read_csv("pokemon.csv")  
  
types = ["Water", "Normal", "Grass", "Bug", "Psychic", "Fire"] # должны быть как в df
water = df.loc[df["Type 1"] == "Water"]["Total"]  
normal = df.loc[df["Type 1"] == "Normal"]["Total"]  
grass = df.loc[df["Type 1"] == "Grass"]["Total"]  
bug = df.loc[df["Type 1"] == "Bug"]["Total"]  
psych = df.loc[df["Type 1"] == "Psychic"]["Total"]  
fire = df.loc[df["Type 1"] == "Fire"]["Total"]  
  
results = stats.f_oneway(water, normal, grass, bug, psych, fire) 
  
type_pairs = []  
for typeA in range(4): # кол-во групп минус 1 (?)
    for typeB in range(typeA+1,5): # кол-во групп плюс 1 (?)
        type_pairs.append((types[typeA], types[typeB]))  
for typeA, typeB in type_pairs:  
    print(typeA, typeB)  
    print(stats.ttest_ind(df.loc[df["Type 1"] == typeA]["Total"],  
                          df.loc[df["Type 1"] == typeB]["Total"]))

# важно - Bonferroni correction
# делим статистическую значимость на количество сравнений. В нашем случае 10
# 0.05 / 10 = 0.005
# значит, ищем pvalue меньше 0.005

# ИЛИ TUKEY TEST

from statsmodels.stats.multicomp import pairwise_tukeyhsd  
  
excluded = [group for group in list(df["Type 1"].value_counts().index) if group not in types]  
df = df[~df["Type 1"].isin(excluded)]  
  
tukey = pairwise_tukeyhsd(endog=df["Total"],  
                          groups=df["Type 1"],  
                          alpha=0.05)  
tukey.plot_simultaneous()  
plt.vlines(x=df["Total"].mean(), ymin=-10, ymax=10, color="red")  
plt.tight_layout()  
plt.show()  
print(tukey.summary())
```

### Alexander-Govern 
```Python
# проверка гомогенности дисперсий
type_pairs = []  
for typeA in range(4):  # кол-во групп минус 1 (?)  
    for typeB in range(typeA + 1, 5):  # кол-во групп плюс 1 (?)  
        type_pairs.append((types[typeA], types[typeB]))  
for typeA, typeB in type_pairs:  
    print(typeA, typeB)  
    print(stats.levene(df.loc[df["Type 1"] == typeA]["Total"],  
                          df.loc[df["Type 1"] == typeB]["Total"]))

stats.alexandergovern(water, normal, grass, bug, psych, fire)
```


### Kruskal-Wallis #H-тест 
```Python
stats.kruskal(water, normal, grass, bug, psych, fire) 
```

### ANOVA тест для зависимых выборок, односторонняя (одна доп. переменная)
Можно многостороннюю, если передать в within список переменных
- Требует более 3 (?) совокупностей
```Python
from` `statsmodels.stats.anova` `import` `AnovaRM`

df = pd.read_csv("rmAOV1way.csv")  
anova = AnovaRM(df, depvar="rt", subject="Sub_id", within=["cond"])  
# depvar = зависимый показатель, который отличается со временем
# subject = id субъектов эксперимента. Всё в одном df, поэтому эти id повторяются
# within = вещь, которая делает анову ановой. Дополнительные переменные, на которые мы смотрим
result = anova.fit()
```

### Friedman для нескольких показателей зависимых непараметрических выборок. Разница в том, что тут используются ранги
- Минимум 3 совокупности
- Требует более 6 совокупностей для большой надёжности
```Python
df = pd.read_csv("rmAOV1way.csv")  
sample1 = df[:60]  
sample2 = df[60:]  
  
result = stats.friedmanchisquare(sample1["rt"], sample2["rt"], sample2["rt"])
# повторил выборок, потому что в df только две
```
