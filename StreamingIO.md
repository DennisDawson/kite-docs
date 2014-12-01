---
layout: page
title: Streaming Input and Output
---

This example how you can use the Kite Data API for performing streaming writes to (and reads from) a dataset.

## Product.class

`Product.class` defines a record with two fields that represent a product: _name_ and _id_. The `MoreObjects.toStringHelper()` method is a fluid builder that makes it easier to construct and return a string containing all of the fields and their values.

```Java

package org.kitesdk.examples.data;

import com.google.common.base.MoreObjects;

/**
 * A POJO representing a product.
 */

public class Product {
  private String name;
  private long id;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public long getId() {
    return id;
  }

  public void setId(long id) {
    this.id = id;
  }

  @Override
  public String toString() {
    return MoreObjects.toStringHelper(this)
        .add("name", name)
        .add("id", id)
        .toString();
  }
}
```

## CreateProductDatasetPojo.java





```
mvn exec:java -Dexec.mainClass="org.kitesdk.examples.data.CreateProductDatasetPojo"
```

You can look at the files that were created in
[`/tmp/data/products`](http://localhost:8888/filebrowser/#/tmp/data/products).

Once we have created a dataset and written some data to it, the next thing to do is to
read it back. We can do this with the `ReadProductDatasetPojo` program.

```
mvn exec:java -Dexec.mainClass="org.kitesdk.examples.data.ReadProductDatasetPojo"
```

Finally, delete the dataset:

```
mvn exec:java -Dexec.mainClass="org.kitesdk.examples.data.DeleteProductDataset"
```
 