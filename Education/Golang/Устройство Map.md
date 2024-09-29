insert, remove, lookup(поиск) - по ключу
все операции за o(1)

простая реализация Map, но работает за o(n)
```
type entry struct {
	key string
	value int
}

type simpleMap []entry

func lookup(m simpleMap, key, string) int {
	for _, e := range m {
		if e.key == key {
			return value
		}
	}
	
	return 0_
}
```

чтобы сделать лучше, нужно разбить все на бакеты, но если используем, к примеру, ссылки, то все элементы могут храниться в одном бакете

### Хэш-функция

bucket = hash(key)
- равномерность
- быстрота
- детерминированность
- криптоустойчивость (нельзя, чтобы кто-то подобрал все ключи для помещения данных в один бакет)

v = m[k] -> v = runtime.lookup(m, k)

Сигнатура lookup:
func <K, V> lookup(m map[K]V, k K) V ?дженерики?

Как обойтись без дженериков?

- Все операции выполняются с помощью unsafe.Pointers (указывает на любой тип данных)
- Мета-информация о типе хранится в type descriptor (хранится инф-ия о типе)
- type descriptor предоставляет операции hash, equal, copy (хранится в дескрипторе типа)

 v = m[k]
 Скомпилируется в : 
 ```
pk := unsafe.Pointer(&k)

func lookup(t *mapType,
			m *mapHeader,
			k unsafe.Pointer) unsafe.Pointer

pv := runtime.lookup(typeOf(m), m, pk)

v = *(*V)pv
 ```

header(map):
size, log(buckets count), \*Bucket (Lob Hash -> Bucket, Log Hash -> Bucket e.t.c), Hash Seed

Low order bits (LOB)
hash(key) = 5461 по 4 бакетам
5461 % 4 = 1
hash(key) = 10101010101*01*
log2(buckets)
2^n = buckets -> 2^2 = 4
mod = 2 -> n - количество младших битов - остаток

LOB(Hash) = *01*
![[Pasted image 20240823221654.png]]
*в каждом бакете не более 8 значений*

все связано с выравниванием типов












