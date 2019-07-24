# ajou-dirtycode refactoring

> 이번 과제는 dirtySample의 전체코드를 분석하고 테스트 코드를 작성했다. 이후에 리팩토링을 진행하였다.


## 전체 코드 파악하기

### 전체 흐름

DirtySample의 `updateQuality()` 메서드는 크게 3개의 if문으로 구성된다.  
시간이 흐르면서 각 item의 quality와 sellIn이 변하고 이에 따라 그 다음 item의 quality가 update된다. 


1) quality의 값 증가 또는 감소 (또는 0)
    - quality의 값은 0 이상 50 이하이다.
2) sellIn의 값 감소 

### item 필드

1. `name`: item 종류
    - `Aged Brie`
    - `Backstage passes to TAFKAL80ETC concert` (*이하 Backstage로 생략*)
    - `Sulfuras, Hand of Ragnaros` (*이하 Sulfuras로 생략*)
    - 세가지 모두 아닌 경우
    
2. `quality`: item 품질

3. `sellIn`: item 판매량


## 조건문 분석하기

### `if`문 -- (1)

```java
if (!items[i].name.equals("Aged Brie") 
    && !items[i].name.equals("Backstage passes to a TAFKAL80ETC concert")) {
    if (items[i].quality > 0) {
        if (!items[i].name.equals("Sulfuras, Hand of Ragnaros")) {
            items[i].quality = items[i].quality - 1;
        }
    }
} else {
    if (items[i].quality < 50) {
        items[i].quality = items[i].quality + 1;

        if (items[i].name.equals("Backstage passes to a TAFKAL80ETC concert")) {
            if (items[i].sellIn < 11) {
                if (items[i].quality < 50) {
                    items[i].quality = items[i].quality + 1;
                }
            }

            if (items[i].sellIn < 6) {
                if (items[i].quality < 50) {
                    items[i].quality = items[i].quality + 1;
                }
            }
        }
    }
}
```

#### 구조 분석

```java
if (name이 Aged Brie도 아니고, Backstage도 아니면) {
    if (quality가 0보다 크다면){
        if (name이 Sulfuras가 아니면){
            item.quality--;
        }
    }
} else { // name이 Aged Brie 또는 Backstage 둘 중 하나
    if (quality가 50보다 작으면) {
        item.quality++;
        if (name이 Backstage 일 때) {
            if (sellIn이 11보다 작은 경우) {
                if (quality가 50보다 작다면) {
                    item.quality++;
                }
            }
            if (sellIn이 6보다 작은 경우) {
                if (quality가 50보다 작다면) {
                    item.quality++;
                }
            }
        }
    }
}

```


#### 분석 내용 정리

- `Aged Brie`:
    1. quality가 50보다 작다면 -> **quality**를 1만큼 **증가**시킨다.

- `Backstage`:
    1. quality가 50보다 작다면 -> **quality**를 1만큼 **증가**시킨다.
    2. sellIn이 11보다 작고 quality가 50보다 작으면 -> **quality**를 1만큼 **증가**시킨다.
    3. sellIn이 6보다 작고 quality가 50보다 작으면 -> **quality**를 1만큼 **증가**시킨다.

- `그 외`:
    1. quality가 0보다 크면 -> **quality**를 1만큼 **감소**시킨다.


### `if`문 -- (2)

```java
if (!item.name.equals("Sulfuras, Hand of Ragnaros")) {
    item.sellIn = item.sellIn - 1;
}
```

#### 구조 분석

```java
if (name이 Sulfuars가 아닐 때)
    item.sellIn--;
```

#### 분석 내용 정리

- `Aged Brie`: **sellIn**을 1만큼 **감소**시킨다.

- `Backstage`: **sellIn**을 1만큼 **감소**시킨다.

- `그 외`: **sellIn**을 1만큼 **감소**시킨다.


### `if`문 -- (3)

```java
if (item.sellIn < 0) {
    if (!item.name.equals("Aged Brie")) {
        if (!item.name.equals("Backstage passes to a TAFKAL80ETC concert")) {
            if (item.quality > 0) {
                if (!item.name.equals("Sulfuras, Hand of Ragnaros")) {
                    item.quality = item.quality - 1;
                }
            }
        } else {
            item.quality = item.quality - item.quality;
        }
    } else {
        if (item.quality < 50) {
            item.quality = item.quality + 1;
        }
    }
}
```

#### 구조 분석

```java
if (sellIn이 0보다 작은 경우) {
    if (name이 Aged Brie가 아닌 경우) {
        if (name이 Backstage가 아닌 경우) {
            if (quality가 0보다 큰 경우) {
                if (name이 Sulfuras가 아닐 때) {
                    item.quality--;
                }
            }
        } else { // name이 Backstage일 때
            item.quality = 0;
        }
    } else { // name이 Aged Brie인 경우
        if (quality가 50보다 작다면) {
            item.quality++;
        }
    }
}
```

#### 분석 내용 정리

- `Aged Brie`:  
    1. sellIn이 0보다 작고 quality 가 50보다 작으면 -> **quality**를 1만큼 **증가**시킨다.

- `Backstage`:  
    1. sellIn이 0보다 작으면 -> **quality**는 **0**이 된다.

- `그 외`: 
    1. sellIn이 0보다 작고 quality 가 0보다 크면 -> **quality**를 1만큼 **감소**시킨다.


### `if`문 --(1)(2)(3) 정리

첫번째 if문 부터 차근차근 내려오면서 살펴보자.  
    
#### - Aged Brie   
1) `quality < 50` -> **quality 1 증가**  
2) **sellIn 1 감소**  
3) `sellIn < 0` && `quality < 50` -> **quality 1 증가**
    

#### - Backstage    
1) `quality < 50`-> **quality 1 증가**
2) `sellIn < 11` && `quality < 50` -> **quality 1 증가**
3) `sellIn < 6` && `quality < 50` -> **quality 1 증가**
4) **sellIn 1 감소**  
5) `sellIn < 0` -> **quality는 0** 

#### - Sulfuras    
~~놀랍게도 아무일도 하지 않는다~~

#### - 그 외 ( ~~Aged Brie~~, ~~BackStage~~, ~~Sulfuras~~)   

1) `quality > 0` -> **quality 1 감소**  
2) **sellIn 1 감소**  
3) `sellIn < 0` && `quality > 0` -> **quality 1 감소**


## test method 구현하기

위에서 name은 크게 4가지로 나뉜다는 점을 다시 한 번 상기하자. test method의 이름과 method 내 코드에서는 4가지 아이템을 앞글자를 따서 간소화하여 표현하였다. 

``` java
private static final String A = "Aged Brie";
private static final String B = "Backstage passes to a TAFKAL80ETC concert";
private static final String S = "Sulfuras, Hand of Ragnaros";
private static final String E = "else..";
```

앞서 크게 나눴던 4가지의 상황에서 quality와 sellIn의 값이 어떻게 변하는지 
hamcrest의 assertThat을 이용하여 확인해보면서 테스트 코드를 작성 하였다.    


## Refactoring

### 1. 복잡한 코드의 간소화

intelliJ가 제공하는 툴을 사용해 간편하게 코드를 리팩토링하는 작업을 진행했다.

#### foreach
가독성을 높이기 위해 for 문을 인덱스 형식 `for(int i=0; i<items.size();i++)` 에서 배열의 모든 요소를 출력하는 **향상된 for문 형식** `for(Item item : items)`으로 변경했다.

![](https://user-images.githubusercontent.com/38287485/61707834-4ff21980-ad86-11e9-9939-7ee631bd0696.png)

#### Invert if condition

조건을 반대로 했을 때 더 이해하기 쉬운 조건식이 있어 리팩토링을 하고 코드를 이해하는 과정을 진행했다.

![](https://user-images.githubusercontent.com/38287485/61707840-51234680-ad86-11e9-942c-f09d50ac429c.png)

#### Merge nested ifs

```java
if (item.sellIn < 11) {
    if (item.quality < 50) {
        item.quality = item.quality + 1;
    }
}
```
위의 원본 코드와 같이 else가 없는 if문이 중복되어 있는 경우,  
두 조건을 merge해 가독성을 높이는 과정을 진행했다.

![](https://user-images.githubusercontent.com/38287485/61708197-2c7b9e80-ad87-11e9-8308-879ed65108e8.png)

```java
if (item.sellIn < 11 && item.quality < 50) {
    item.quality = item.quality + 1;
}
```

#### Simplify

```java
item.quality = item.quality - item.quality;
```
위의 경우 대입하는 값이 항상 0이기 때문에 `item.quality = 0;`으로 변경해주어 가독성을 높였다.

![](https://user-images.githubusercontent.com/38287485/61708747-7fa22100-ad88-11e9-8201-4162d4d0323e.png)


### 2. 불명확한 코드를 명확하게 변경

#### Rename

- `DirtySample` 클래스 이름 변경:  
시간이 흐르면서 각 item의 quality와 sell In이 변하고 이에 따라 그 다음 item의 quality를 update하는 코드의 전체 흐름을 반영하기 위해   
클래스 이름은 DirtySample에서 조금 더 의미 있는 **`ManageItem`** 으로 바꾸었다. 

- `updateQuality()` 메서드 이름 변경:  
item의 quality 필드와 함께 sellIn 필드도 변하기 때문에 **`updateItemFields()`** 로 바꾸었다.


### 3. 중복 코드 제거

#### extract variable   

문자열 상수가 반복되어 변수로 지정하여 재사용할 수 있도록 extract 했다.

```java
private static final String AGED_BRIE = "Aged Brie";
private static final String BACKSTAGE = "Backstage passes to a TAFKAL80ETC concert";
private static final String SULFURAS = "Sulfuras, Hand of Ragnaros";
```

#### extract method

전체 코드 흐름 파악에서 미리 말했듯이 해당 메서드는 크게 if문 3개로 나누어져 있다.

`if else`문 보다 가독성이 높은 `switch case`문을 사용해 item name 4가지 case에 따라 중복되는 코드들을 추출하는 과정을 진행했다.  

``` java
void updateItemFields() {
    for (Item item : items) {
        switch (item.name) {
            case AGED_BRIE:
                updateAgedBrieFields(item);
                break;
            case BACKSTAGE:
                updateBackstageFields(item);
                break;
            case SULFURAS:
                break;
            default:
                updateElseFields(item);
                break;
        }
    }
}
```

위에서 했던 if문 --(1)(2)(3)을 분석하는 과정에서 중복되는 조건들을 찾아내었고, 이 중복 코드를 method로 extract해 재사용하여 간결하고 가독성을 높일 수 있었다. 

1. **quality가 50보다 작으면 1 증가**:  
    ```java
    private void increaseQualityLowerThanFifty(Item item) {
        if (item.quality < 50)
            item.quality++;
    }
    ```
    - Aged Brie일 경우
        ```java
        private void updateAgedBrieFields(Item item) {
            increaseQualityLowerThanFifty(item);
            ...
            if (item.sellIn < 0) increaseQualityLowerThanFifty(item);
        }
        ```

    - Backstage일 경우
        ```java
        private void updateBackstageFields(Item item) {
            increaseQualityLowerThanFifty(item);
            if (item.sellIn < 11) increaseQualityLowerThanFifty(item);
            if (item.sellIn < 6) increaseQualityLowerThanFifty(item);
            ...
        }
        ```
2. **quality가 0보다 크면 1 감소**:
    ```java
    private void decreaseQualityHigherThanZero(Item item) {
        if (item.quality > 0)
            item.quality--;
    }
    ```
    - 그 이외의 경우
        ```java
        private void updateElseFields(Item item) {
            decreaseQualityHigherThanZero(item);
            ...
            if (item.sellIn < 0) decreaseQualityHigherThanZero(item);
        }
        ```

3. **sellIn 1 감소**:
    ```java
    private void decreaseSellIn(Item item) {
        item.sellIn--;
    }
    ```
    - Aged Brie일 경우
        ```java
        private void updateAgedBrieFields(Item item) {
            ...
            decreaseSellIn(item);
            ...
        }
        ```
    - Backstage일 경우
        ```java
        private void updateBackstageFields(Item item) {
            ...
            decreaseSellIn(item);
            ...
        }
        ```
    - 그 이외의 경우
        ```java
        private void updateElseFields(Item item) {
            ...
            decreaseSellIn(item);
            ...
        }
        ```


