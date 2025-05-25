# Лабораторная работа №3: Структруы данных
Есть министерство из N чиновников, где N натуральное число. У каждого из чиновников могут быть как подчиненные, так и начальники, причем справедливы правила: подчиненные моего подчиненного мои подчиненные, начальники моего начальника - мои начальники, мой начальник не есть мой подчиненный, у каждого чиновника не более одного непосредственного начальника.
Для того чтобы получить лицензию на вывоз меди необходимо получить подпись 1-ого чиновника - начальника всех чиновников. Проблема осложняется тем, что каждый чиновник, вообще говоря, может потребовать "визы", т.е. подписи некоторых своих непосредственных подчиненных и взятку - известное количество долларов. Для каждого чиновника известен непустой список возможных наборов "виз" и соответствующая каждому набору взятка. Пустой набор означает, что данный чиновник не требует виз в данном случае. Чиновник ставит свою подпись лишь после того, как ему представлены все подписи одного из наборов "виз" и уплачена соответствующая взятка.
Необходимо определить и вывести минимальный по сумме уплаченных взяток допустимый порядок получения подписей для лицензии и стоимость.
N<100. Количество наборов для каждого чиновника не превосходит 15.
## Реализация
### Листинг программы:
``` python
import time
from collections import defaultdict

class ArrayBureaucracy:
    def __init__(self, n):
        self.chiefs = [None] * n
        self.subordinates = [[] for _ in range(n)]
        self.bribes = [defaultdict(int) for _ in range(n)]

class LinkedListNode:
    def __init__(self, id):
        self.id = id
        self.chief = None
        self.subordinates = []
        self.bribes = {}

class LinkedListBureaucracy:
    def __init__(self, n):
        self.officials = [LinkedListNode(i) for i in range(n)]

class StdLibBureaucracy:
    def __init__(self, n):
        self.chiefs = {}
        self.subordinates = defaultdict(list)
        self.bribes = defaultdict(dict)

def main():
    print("Автор: Анаприенко Даниил Сергеевич")
    print("Группа: АИСа-о24\n")
    
    # Пример входных данных
    N = 5
    hierarchy = [None, 0, 0, 1, 1]  # начальники
    bribes = {
        0: {frozenset({1, 2}): 100, frozenset({1}): 200},
        1: {frozenset({3, 4}): 50, frozenset({3}): 80},
        2: {frozenset(): 30},
        3: {frozenset(): 20},
        4: {frozenset(): 10}
    }
    
    def test_implementation(impl_class, name):
        print(f"\nТестирование реализации: {name}")
        start_time = time.perf_counter()
        
        bureau = impl_class(N)
        
        # Строим иерархию
        for sub in range(N):
            chief = hierarchy[sub]
            if chief is not None:
                if isinstance(bureau, ArrayBureaucracy):
                    bureau.chiefs[sub] = chief
                    bureau.subordinates[chief].append(sub)
                elif isinstance(bureau, LinkedListBureaucracy):
                    bureau.officials[sub].chief = bureau.officials[chief]
                    bureau.officials[chief].subordinates.append(bureau.officials[sub])
                else:
                    bureau.chiefs[sub] = chief
                    bureau.subordinates[chief].append(sub)
        
        # Добавляем взятки
        for official in bribes:
            for visa_set, bribe in bribes[official].items():
                if isinstance(bureau, ArrayBureaucracy):
                    bureau.bribes[official][visa_set] = bribe
                elif isinstance(bureau, LinkedListBureaucracy):
                    bureau.officials[official].bribes[visa_set] = bribe
                else:
                    bureau.bribes[official][visa_set] = bribe
        
        memo = {}
        
        def dfs(official):
            if official in memo:
                return memo[official]
            
            if isinstance(bureau, LinkedListBureaucracy):
                subs = [node.id for node in bureau.officials[official].subordinates]
                current_bribes = bureau.officials[official].bribes
            else:
                subs = bureau.subordinates[official]
                current_bribes = bureau.bribes[official]
            
            if not subs:
                memo[official] = (0, [official])
                return (0, [official])
            
            min_cost = float('inf')
            best_path = []
            
            for visa_set, bribe in current_bribes.items():
                total_cost = bribe
                current_path = []
                valid = True
                for sub in visa_set:
                    if sub not in subs:
                        valid = False
                        break
                    cost, path = dfs(sub)
                    total_cost += cost
                    current_path.extend(path)
                if not valid:
                    continue
                
                if total_cost < min_cost:
                    min_cost = total_cost
                    best_path = current_path.copy()
                    best_path.append(official)
            
            memo[official] = (min_cost, best_path)
            return (min_cost, best_path)
        
        total_cost, path = dfs(0)
        elapsed = time.perf_counter() - start_time
        
        print(f"Минимальная стоимость: {total_cost}")
        print(f"Порядок получения подписей: {path}")
        print(f"Время выполнения: {elapsed:.6f} сек")
        
        return elapsed
    
    # Тестирование всех реализаций
    results = {
        "Массив": test_implementation(ArrayBureaucracy, "На массивах"),
        "Связанный список": test_implementation(LinkedListBureaucracy, "На связанных списках"),
        "Стандартные структуры": test_implementation(StdLibBureaucracy, "С использованием стандартных структур")
    }
    
    # Сравнение
    print("\nСравнение производительности:")
    fastest = min(results.values())
    for name, time_spent in results.items():
        if fastest == 0:
            ratio = "∞"
        else:
            ratio = f"{time_spent / fastest:.1f}"
        print(f"{name}: {time_spent:.6f} сек (в {ratio} раз медленнее)")

if __name__ == "__main__":
    main()
```
# Результат выполнения программы:
![image](https://github.com/user-attachments/assets/1e0f2d9a-e042-4784-a812-72417902680b)

