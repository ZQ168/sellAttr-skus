# sellAttr-skus
# 电商平台基于销售属性生成SKU的设计
电商系统中商品系统**最重要**的功能就是SKU的创建以及维护，所以一旦出错就会影响电商后面的流程。
## 设计
业务流程，如图通过设置销售属性值销售属性组来实现组合skus的实现

![销售属性后管效果](https://user-images.githubusercontent.com/28287510/159254765-018a88ed-951c-4e16-9b06-e490c7d29f44.png)
业务总体设计  
![组合之后的图片](https://user-images.githubusercontent.com/28287510/159257429-5eb08ca0-c58e-484f-a2ce-01fc5d5b7216.png)  
![销售属性重构总方案](https://user-images.githubusercontent.com/28287510/159255137-05dafb92-fd52-4d0b-84fb-9c353d34aee2.png)
### SKU创建

> * 新增、删除、修改属性值
> * 删掉的销售属性不能够被重新生成出来
> * 勾选属性值、取消勾选属性值  

### 1. 销售属性组合
笛卡尔积组合
```javascript
calcDescartes(array) {
  return [].reduce.call(array, function (col, set) {
    let res = [];
    col.forEach(function (colItem) {
      set.forEach(function (setItem) {
        let newArr = [].concat(Array.isArray(colItem) ? colItem : [colItem]);
        newArr.push(setItem);
        res.push(newArr);
      });
    });
    return res;
  });
}
```
改成以属性组名称和属性值作为判断，通过map结构存储，键值为属性组名称和属性名称组合成为唯一attrValId

全排列组合，每一个组合与sku map结构对比，包含（新增维度）被包含（删除维度，删除属性值）命中（新增属性，编辑属性值）

生成一个唯一标识的attrValId

同一维度属性值不能重复

属性组名称不能重复

### 2. 被删除的skus不能够被重新生成出来   
![被删除的skus不能够被重新生成出来](https://user-images.githubusercontent.com/28287510/159255064-c148ea3e-4b89-4b18-a44b-3430b7cd9215.png)  
1、记录上一次的销售属性值，上一次销售属性（笛卡尔积）组合之后的属性值唯一标识attrValId组合

2、上一次的skus，上一次的sku列表map结构生成

3、map.keys()与销售属性全排列组合对比，对比差异（全等）取差集

4、当前的销售属性进行笛卡尔积，遍历

5、差集不生成sku


### 3. 取消勾选的属性值不能够被重新生成出来  
判断是不是取消勾选操作，取消勾选之后再次勾选

取消勾选：cancelSkusMap添加当前map键为id，值为上一次的skusMap包含当前id的skus信息数组

取消勾选之后再次勾选：cancelSkusMap中是否有数据，并且当前不是只有一条数据被checked（最后一条数据不能用取消勾选的数据，要保留skuId原则）

![取消重新勾选](https://user-images.githubusercontent.com/28287510/159255158-c7e53085-b6c0-49cd-93ce-6e54e861fa7c.png)



