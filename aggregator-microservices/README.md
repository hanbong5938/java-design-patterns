---
title: Aggregator Microservices
category: Architectural
language: en
tag:
- Cloud distributed
- Decoupling
- Microservices
---
## 설명
여러 개의 다른 마이크로 서비스에서 데이터를 수집하여 하나의 응답으로 결합하는 타입의 마이크로 서비스이다.
여러 마이크로 서비스에 흩어져 있는 데이터를 통합적으로 보여줄 수 있으며, 혹은 응답을 반환하기 전에 데이터를 처리하거나 집계할 수도 있다.
클라이언트의 요청 수를 줄이고 데이터 처리를 단순화 할 수 있으며, 집계 로직을 별도의 마이크로 서비스로 분리하면 다른 서비스와는 독립적으로 스케일링 할 수 있고, 다수의 애플리케이션에서 재사용 할 수 있다.


## Intent

The user makes a single call to the aggregator service, and the aggregator then calls each relevant microservice.

## Explanation

Real world example

> Our web marketplace needs information about products and their current inventory. It makes a call to an aggregator
> service which in turn calls the product information microservice and product inventory microservice returning the
> combined information. 

In plain words

> Aggregator Microservice collects pieces of data from various microservices and returns an aggregate for processing. 

Stack Overflow says

> Aggregator Microservice invokes multiple services to achieve the functionality required by the application.

**Programmatic Example**

Let's start from the data model. Here's our `Product`.

```java
public class Product {
  private String title;
  private int productInventories;
  // getters and setters ->
  ...
}
```

Next we can introduce our `Aggregator` microservice. It contains clients `ProductInformationClient` and
`ProductInventoryClient` for calling respective microservices.

```java
@RestController
public class Aggregator {

  @Resource
  private ProductInformationClient informationClient;

  @Resource
  private ProductInventoryClient inventoryClient;

  @RequestMapping(path = "/product", method = RequestMethod.GET)
  public Product getProduct() {

    var product = new Product();
    var productTitle = informationClient.getProductTitle();
    var productInventory = inventoryClient.getProductInventories();

    //Fallback to error message
    product.setTitle(requireNonNullElse(productTitle, "Error: Fetching Product Title Failed"));

    //Fallback to default error inventory
    product.setProductInventories(requireNonNullElse(productInventory, -1));

    return product;
  }
}
```

Here's the essence of information microservice implementation. Inventory microservice is similar, it just returns
inventory counts.

```java
@RestController
public class InformationController {
  @RequestMapping(value = "/information", method = RequestMethod.GET)
  public String getProductTitle() {
    return "The Product Title.";
  }
}
```

Now calling our `Aggregator` REST API returns the product information.

```bash
curl http://localhost:50004/product
{"title":"The Product Title.","productInventories":5}
```

## Class diagram

![alt text](./aggregator-service/etc/aggregator-service.png "Aggregator Microservice")

## Applicability

Use the Aggregator Microservices pattern when you need a unified API for various microservices, regardless the client device.

## Credits

* [Microservice Design Patterns](http://web.archive.org/web/20190705163602/http://blog.arungupta.me/microservice-design-patterns/)
* [Microservices Patterns: With examples in Java](https://www.amazon.com/gp/product/1617294543/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=javadesignpat-20&creative=9325&linkCode=as2&creativeASIN=1617294543&linkId=8b4e570267bc5fb8b8189917b461dc60)
* [Architectural Patterns: Uncover essential patterns in the most indispensable realm of enterprise architecture](https://www.amazon.com/gp/product/B077T7V8RC/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=javadesignpat-20&creative=9325&linkCode=as2&creativeASIN=B077T7V8RC&linkId=c34d204bfe1b277914b420189f09c1a4)
