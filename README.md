# `append`と`extend`の違いを知らないと、思わぬバグを生むことになる

![](https://raw.githubusercontent.com/yKesamaru/append_and_extend/master/assets/g4597.png)

Pythonにおける`append`と`extend`について、使い方を誤ると思わぬバグを生むことになります。この記事では`append`と`extend`の基本的な使い方と、面倒くさいことを考えずに済むように、ラッパー関数を作成し、最後に再利用可能なモジュール化を行います。

## 基本事項
### `append`と`extend`の基本的な動作

- `append`: リストの末尾に新しい要素を追加します。
- `extend`: リストの末尾にイテラブルなオブジェクトの各要素を追加します。

`append`と`extend`の使い方を間違えると、意図しない結果やエラーが発生する可能性があります。以下に例を示します。


### 1. リストに要素を追加する場合

#### 正しい使い方:
```python
lst = [1, 2, 3]
lst.append(4)  # [1, 2, 3, 4]
```
この場合、`append`はリストの末尾に新しい要素を追加します。

#### 誤用:
```python
lst = [1, 2, 3]
lst.extend(4)  # TypeError: 'int' object is not iterable
```
この場合、`extend`はイテラブルなオブジェクトを期待していますが、整数が渡されたためエラーが発生します。

### 2. リストにリストを追加する場合

#### 正しい使い方:
```python
lst1 = [1, 2, 3]
lst2 = [4, 5, 6]
lst1.extend(lst2)  # [1, 2, 3, 4, 5, 6]
```
この場合、`extend`は`lst2`の要素を`lst1`の末尾に追加します。

#### 誤用:
```python
lst1 = [1, 2, 3]
lst2 = [4, 5, 6]
lst1.append(lst2)  # [1, 2, 3, [4, 5, 6]]
```
この場合、`append`は`lst2`をそのまま`lst1`の末尾に追加し、ネストされたリストができます。

### 3. 上記をforループで使う場合

#### 正しい使い方:
```python
lst1 = [1, 2, 3]
lst2 = [4, 5, 6]
for item in lst2:
    lst1.append(item)  # [1, 2, 3, 4, 5, 6]
```

#### 誤用:
```python
lst1 = [1, 2, 3]
lst2 = [4, 5, 6]
for item in lst2:
    lst1.extend(item)  # TypeError: 'int' object is not iterable
```
この場合、`extend`はイテラブルなオブジェクトを期待していますが、`lst2`の各要素は整数であるためエラーが発生します。


### 4. `np.ndarray`をリストに追加する場合

#### 正しい使い方:
```python
lst = [1, 2, 3]
arr = np.array([4, 5, 6])
lst.append(arr)  # [1, 2, 3, array([4, 5, 6])]
```
この場合、`append`は`arr`をそのまま`lst`の末尾に追加し、リストの中に`np.ndarray`が含まれる形になります。

#### 誤用:
```python
lst = [1, 2, 3]
arr = np.array([4, 5, 6])
lst.extend(arr)  # [1, 2, 3, 4, 5, 6]
```
この場合、`extend`は`arr`の各要素を`lst`の末尾に追加します。しかし、この結果はエラーではありませんが、`np.ndarray`をそのままリストに追加する意図とは異なる結果になります。

### 5. `np.ndarray`同士を結合する場合

#### 正しい使い方:
```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
combined = np.concatenate((arr1, arr2))  # array([1, 2, 3, 4, 5, 6])
```
この場合、`np.concatenate`を使用して、2つの`np.ndarray`を結合します。

#### 誤用:
`np.ndarray`同士の結合には、基本的に`np.concatenate`を使います。`append`や`extend`を使用すると、ネストされた配列になったり、型の不一致から予期しない「自動変換」が起こる可能性があります。
以下に例を示します。




### `np.ndarray`をリストに追加する場合の例

```python
import numpy as np

lst = ['舞浜', 100]
arr = np.array([200, 300])
```

#### `append`を使用する場合:
```python
lst.append(arr)
print(lst)  # ['舞浜', 100, array([200, 300])]
```
`append`を使用すると、`arr`という`np.ndarray`オブジェクト自体がリストの新しい要素として追加されます。その結果、リストの中に`np.ndarray`がネストされた形になります。

#### `extend`を使用する場合:
```python
lst.extend(arr)
print(lst)  # ['舞浜', 100, 200, 300]
```
`extend`を使用すると、`arr`の各要素（この場合は200と300）がリストに追加されます。その結果、リストは`np.ndarray`の要素をフラットに結合した形になります。

### なぜ異なる結果になるのか

`append`は与えられたオブジェクトをそのままリストに追加するのに対し、`extend`は与えられたイテラブルなオブジェクトの各要素をリストに追加します。そのため、`np.ndarray`をリストに追加する際に`extend`を使用すると、`np.ndarray`の各要素がリストに直接追加されるため、意図しない結果になります。

## 問題
問題は、コード中にリスト追加部分が多くあり、`append`と`extend`の使い分けが難しい場合です。

## 解決法
このような場合は、毎度`if`文で`append`と`extend`を使い分けるのは面倒です。そこで、`append`と`extend`をラップした関数を作成し、`append`と`extend`の使い分けを関数内で行うことで、コードを簡潔にできます。

### ラッパー関数その１
```python
import numpy as np

def smart_combine(a, b):
    # 両方がnp.ndarrayの場合
    if isinstance(a, np.ndarray) and isinstance(b, np.ndarray):
        return np.concatenate((a, b))
    
    # aがリストで、bがイテラブルなオブジェクト（文字列を除く）の場合
    elif isinstance(a, list) and hasattr(b, '__iter__') and not isinstance(b, str):
        if isinstance(b, np.ndarray):
            a.append(b)  # bがnp.ndarrayの場合、リストに追加
        else:
            a.extend(b)  # それ以外の場合、要素をリストに追加
        return a
    
    # bがリストで、aがイテラブルなオブジェクト（文字列を除く）の場合
    elif isinstance(b, list) and hasattr(a, '__iter__') and not isinstance(a, str):
        if isinstance(a, np.ndarray):
            b.append(a)  # aがnp.ndarrayの場合、リストに追加
        else:
            b.extend(a)  # それ以外の場合、要素をリストに追加
        return b
    
    # bがリストで、aが単一の要素の場合
    elif isinstance(b, list):
        b.append(a)
        return b
    
    # それ以外の場合
    else:
        a.append(b)
        return a

```

`smart_combine`は、2つの引数`a`と`b`を取る関数です。この関数の目的は、リストや`np.ndarray`などの異なるデータ型を適切に結合することです。

1. **両方がnp.ndarrayの場合**:
    - `a`と`b`がともに`np.ndarray`の場合、`np.concatenate`関数を使用して2つの配列を結合します。

2. **aがリストで、bがイテラブルなオブジェクト（文字列を除く）の場合**:
    - `b`が`np.ndarray`の場合、`b`をそのまま`a`のリストに追加します。
    - それ以外の場合、`b`の要素を`a`のリストに追加します。

3. **bがリストで、aがイテラブルなオブジェクト（文字列を除く）の場合**:
    - `a`が`np.ndarray`の場合、`a`をそのまま`b`のリストに追加します。
    - それ以外の場合、`a`の要素を`b`のリストに追加します。

4. **bがリストで、aが単一の要素の場合**:
    - `a`をそのまま`b`のリストに追加します。

5. **それ以外の場合**:
    - `b`をそのまま`a`のリストに追加します。

この関数を使用することで、リストや`np.ndarray`を適切に結合できます。`np.ndarray`とリストを組み合わせる場合や、リスト内に`np.ndarray`を追加する場合など、`append`や`extend`をそのまま使うより楽ができます。

### ラッパー関数その２
メモリー効率を考慮すると、`append`や`extend`を使用するよりも、`itertools.chain`を使用した方が良い場合があります。
```python
import numpy as np
from itertools import chain

def smart_combine_with_itertools(a, b):
    # 両方がnp.ndarrayの場合
    if isinstance(a, np.ndarray) and isinstance(b, np.ndarray):
        return np.concatenate((a, b))
    
    # aがリストで、bがイテラブルなオブジェクト（文字列を除く）の場合
    elif isinstance(a, list) and hasattr(b, '__iter__') and not isinstance(b, str):
        if isinstance(b, np.ndarray):
            return list(chain(a, [b]))  # bがnp.ndarrayの場合、リストに追加
        else:
            return list(chain(a, b))  # それ以外の場合、要素をリストに追加
    
    # bがリストで、aがイテラブルなオブジェクト（文字列を除く）の場合
    elif isinstance(b, list) and hasattr(a, '__iter__') and not isinstance(a, str):
        if isinstance(a, np.ndarray):
            return list(chain(b, [a]))  # aがnp.ndarrayの場合、リストに追加
        else:
            return list(chain(b, a))  # それ以外の場合、要素をリストに追加
    
    # bがリストで、aが単一の要素の場合
    elif isinstance(b, list):
        return list(chain(b, [a]))
    
    # それ以外の場合
    else:
        return list(chain(a, [b]))

```
`itertools.chain`を使用すると、ラッパー関数その1と比較して、以下の点で有利です。

1. **効率性**:
   - `itertools.chain`はイテラブルを連鎖的に結合するジェネレーターとして機能するので、メモリ効率が良いです。
   - 一方、`append`や`extend`を使用すると、リストの再確保やコピーが発生します。リストのサイズが大きくなる場合に不利になります。

2. **拡張性**:
   - `itertools.chain`は任意の数のイテラブルを受け取ることができるため、2つ以上のイテラブルを結合する場合にも使用できます。
   - `append`や`extend`は、1つの要素または1つのイテラブルしかリストに追加できません。

3. **シンプルさ**:
   - `itertools.chain`を使用すると、リストや他のイテラブルを結合する操作が一貫してシンプルになります。特定の条件ごとに`append`や`extend`を使い分ける必要がなくなります。

注意点
- `itertools.chain`はジェネレーターを返すため、結果をリストとして取得する場合は、`list()`関数を使用してジェネレーターをリストに変換する必要があります。
- `itertools.chain`を使用する場合、結果のイテラブルは元のリストやイテラブルを変更しません。したがって、元のリストやイテラブルに変更を加える場合は、同じ変数名で上書きする必要があります。

### モジュールとして利用する
複数にまたがるコード中に`append`や`extend`が多用される場合、ラッパー関数をモジュールとして作成しておくと便利です。以下に、ラッパー関数を`combine.py`という名前のモジュールとして作成する例を示します。

```python
from itertools import chain
from typing import List, TypeVar

T = TypeVar('T')
import numpy as np

from face01lib.logger import Logger


class Comb:
    def __init__(self, log_level: str = 'info'):
        # Setup logger: common way
        self.log_level: str = log_level
        import os.path
        name: str = __name__
        dir: str = os.path.dirname(__file__)
        parent_dir, _ = os.path.split(dir)

        self.logger = Logger(self.log_level).logger(name, parent_dir)

    @staticmethod
    def comb(a: List[T], b: List[T]) -> List[T]:
        # 両方がnp.ndarrayの場合
        if isinstance(a, np.ndarray) and isinstance(b, np.ndarray):
            return list(np.concatenate((a, b)))

        # aがリストで、bがイテラブルなオブジェクト（文字列を除く）の場合
        elif isinstance(a, list) and hasattr(b, '__iter__') and not isinstance(b, str):
            if isinstance(b, np.ndarray):
                return list(chain(a, b.tolist()))  # bをリストに変換してから結合
            else:
                return list(chain(a, b))

        # bがリストで、aがイテラブルなオブジェクト（文字列を除く）の場合
        elif isinstance(b, list) and hasattr(a, '__iter__') and not isinstance(a, str):
            if isinstance(a, np.ndarray):
                return list(chain(b, a.tolist()))  # aをリストに変換してから結合
            else:
                return list(chain(b, a))

        # bがリストで、aが単一の要素の場合
        elif isinstance(b, list):
            if isinstance(a, np.ndarray):
                return list(chain(b, a.tolist()))  # aをリストに変換してから結合
            elif isinstance(a, list):
                return list(chain(b, a))  # aがリストの場合、そのまま結合
            else:
                return list(chain(b, [a]))  # aがリストでない場合、リストにしてから結合

        # それ以外の場合
        else:
            return list(chain(a, [b]))
```

以下が使用方法です。
```python
from face01lib.combine import Comb as C

known_face_encodings = C.comb(known_face_encodings, face_encoding_list)
face_file_name_list = C.comb(face_file_name_list, [face_image])
```
上記の例で変数`face_image`は`str`ですが、リスト化することで使用可能です。

以上です。ありがとうございました。