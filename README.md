# ajou-dirtycode refactoring

> 이번 과제는 테스트 코드를 먼저 작성한 후, 리팩토링을 하는 순서로 진행되었다.

<hr/>

## 전체 코드 파악하기

### 전체 흐름

DirtySample의 `updateQuality()` 메서드는 크게 3개의 if문으로 구성된다.  
시간이 흐르면서 각 item의 quality와 sellIn이 변하고 이에 따라 그 다음 item의 quality가 update된다. 


1) quality의 값 증가 또는 감소 (또는 0)
    - quality의 값은 0 이상 50 이하이다.
2) sellIn의 값 감소 

### item 필드

1. **name**: item 종류
    - `Aged Brie`
    - `Backstage passes to TAFKAL80ETC concert` (*이하 `Backstage`로 생략*)
    - `Sulfuras, Hand of Ragnaros` (*이하 `Sulfuras`로 생략*)
    - 세가지 모두 아닌 경우
    
2. **quality**: item 품질

3. **sell In**: item 판매량

<hr/>

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

* **`if`** name이 Aged Brie도 아니고, Backstage도 아닌 경우
    - **`if`** quality가 0보다 큰 경우
        + **`if`** name이 Sulfuras가 아니라면  
        **quality 1 감소**
        
* **`else`** name이 Aged Brie 또는 Backstage 둘 중 하나인 경우
    - **`if`** quality가 50보다 작으면  
    **quality 1 증가**
        + **`if`** name이 Backstage 일 때
            * **`if`** sellIn이 11보다 작은 경우
                - **`if`** quality가 50보다 작다면  
                **quality 1 증가** 
            * **`if`** sellIn이 6보다 작은 경우
                - **`if`** quality가 50보다 작다면  
                **quality 1 증가** 


#### 분석 내용 정리

- **Aged Brie**:
    1. quality가 50보다 작다면  
    -> **quality를 1만큼 증가**시킨다.

- **Backstage**:
    1. quality가 50보다 작다면  
    -> **quality를 1만큼 증가**시킨다.
    2. sellIn이 11보다 작고 quality가 50보다 작으면  
    -> **quality를 1만큼 증가**시킨다.
    3. sellIn이 6보다 작고 quality가 50보다 작으면  
    -> **quality를 1만큼 증가**시킨다.

- **그 외**:
    1. quality가 0보다 크면  
    -> **quality를 1만큼 감소**시킨다.


### `if`문 -- (2)

```java
if (!item.name.equals("Sulfuras, Hand of Ragnaros")) {
    item.sellIn = item.sellIn - 1;
}
```

#### 구조 분석

* **`if`** name이 Sulfuars가 아닐 때  
    - **sellIn 1 감소**

#### 분석 내용 정리

- **Aged Brie**:  
    -> **sellIn을 1만큼 감소**시킨다.

- **Backstage**:  
    -> **sellIn을 1만큼 감소**시킨다.

- **그 외**:  
    -> **sellIn을 1만큼 감소**시킨다.


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

* **`if`** sellIn이 0보다 작은 경우
    - **`if`** name이 Aged Brie가 아닌 경우
        + **`if`** name이 Backstage가 아닌 경우
            * **`if`** quality가 0보다 큰 경우
                - **`if`** name이 Sulfuras가 아닐 때  
                **quality 1 감소**
        + **`else`** name이 Backstage일 때
            **quality는 0**
    - **`else`** name이 Aged Brie인 경우
        + **`if`** quality가 50보다 작다면  
        **quality 1 증가**
 

#### 분석 내용 정리

- **Aged Brie**:  
    1. sellIn이 0보다 작고 quality 가 50보다 작으면  
    -> **quality를 1만큼 증가**시킨다.

- **Backstage**:  
    1. sellIn이 0보다 작으면
    -> **quality는 0**이 된다.

- **그 외**: 
    1. sellIn이 0보다 작고 quality 가 0보다 크면  
    -> **quality를 1만큼 감소**시킨다.


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

<hr/>

## 테스트 method 구현하기

위에서 name은 크게 4가지로 나뉜다는 점을 다시 한 번 상기하자. <br>
그러므로 크게 4가지의 상황에서 quality와 sellIn의 값이 어떻게 변하는지 
hamcrest의 assertThat을 이용하여 확인해보면서 테스트 코드를 작성 하였다.    

<hr/>

## Refactoring

### 1. 클래스 이름 변경

시간이 흐르면서 각 item의 quality와 sell In이 변하고 이에 따라 그 다음 item의 quality를 update하는 코드의 전체 흐름을 반영하기 위해   
클래스 이름은 DirtySample에서 조금 더 의미 있는 **`ManageItem`** 으로 바꾸었다. 

### 2. `updateItem()` 메서드

기존의 `updateQuality()` 메서드에서 item의 sellIn이 변하기 때문에 포괄하기 위해 메서드 이름을 `updateItem()` 로 바꾸었다.

- 가독성을 높이기 위해 for 문을 인덱스 형식 `for(int i=0; i<items.size();i++)` 에서 배열의 모든 요소를 출력하는 **향상된 for문 형식** `for(Item item : items)`으로 변경했다.

- 전체 코드 흐름 파악에서 미리 말했듯이 해당 메서드는 크게 if문 3개로 나누어졌다.  
각 if문에 대해 `updateQualityExceptForSulfuras(item)`, `updateSellIn(item)`, `updateQualityBasedOnSellInLowerThan0(item)` 메서드로 extract하였다.

``` java
public void updateItem() {
	for (Item item :items) {
		updateQualityExceptForSulfuras(item);
		updateSellIn(item);
		updateQualityBasedOnSellInLowerThan0(item);
	}
}
```

### 3. `updateQualityExceptForSulfuras(item)`

Sulfuras를 제외한 item에 대해서 quality를 update하는 메서드이다.

* 

``` java
 private void updateQualityExceptForSulfuras(Item item) {
        if (isExceptForAgedBrieAndBackstage(item)) {
            manageQualityBasedOnNotSulfuras(item);
        } else {
            manageQualityBasedOnAgedBrieOrBackstage(item);
        }
    }
```