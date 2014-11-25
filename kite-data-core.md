---
layout: page
title: kite-data-core
---

The kite-data-core module provides primary support for the most common tasks required to create and maintain Hadoop datasets.

The `Datasets` class provides methods to create, update, and delete datasets.

Equally important is the `DatasetDescriptor` class, which stores the structural definition of a dataset, including its schema and partition strategy. `DatasetDescriptor.Builder` is a fluent builder you can use when defining new datasets.

<a name="Example"/>

## Example: Hello Kite!

`HelloKite` demonstrates how to create a dataset, add a record, read the record, then delete the dataset. 

`Hello.class` defines one field named _person_. 

`MakeDataset.class` performs Create, Add, Retrieve, and Delete tasks on a _hello_ dataset instance.

`HelloKite.class` is an instance of the ToolRunner framework used to run a Hadoop application.

Complete code and commentary follow.

### Hello.class

`Hello.class` defines a [POJO](http://en.wikipedia.org/wiki/Plain_Old_Java_Object) that represents one row in the dataset. The only field in the  `Hello` record is a _person_, used by the sayHello method to concatenate a greeting.

```Java
package org.kitesdk.examples.data;

public class Hello {
  private String person;

  public Hello(String person) {
    this.person = person;
  }
	
  public Hello() {
    // Empty constructor for serialization purposes
  }

  public String getPerson() {
    return person;
  }

  public void setPerson(String person) {
    this.person = person;
  }

  public void sayHello() {
    System.out.println("Hello, " + person + "!");
  }
}
```

### MakeDataset.class

Kite can create a dataset based on the `Hello.class`. 

The `MakeDataset.class` demonstrates the following dataset lifecycle tasks:

1. Infer a schema from a Java class.
1. Create a dataset using that schema.
1. Write a record to the dataset.
1. Read a record from a dataset.
1. Delete the dataset once the example is complete.

Kite creates datasets in Avro format by default, and stores them in HDFS.

Here is the complete listing for `MakeDataset.class` with line-by-line commentary.

```Java
package org.kitesdk.examples.data;

import org.kitesdk.data.Dataset;           // Defines a dataset instance
import org.kitesdk.data.DatasetDescriptor; // Stores dataset metadata
import org.kitesdk.data.DatasetReader;     // Reads records from a dataset
import org.kitesdk.data.DatasetWriter;     // Writes records to a dataset
import org.kitesdk.data.Datasets;          // Provides common dataset management methods

public class MakeDataset {

   public MakeDataset() {
   }
	
   public static void makeDataset() { 

      String datasetUri = "dataset:file:/tmp/hellos";
      String person = "Bob";

   // *** 1. Infer the schema from Hello.class. ***

   // DatasetDescriptor.Builder().schema() scans `Hello.class` and extracts
   // the name and type of each defined field. In this example, there is only
   // one defined field, named _person_.

      DatasetDescriptor descriptor = new DatasetDescriptor.Builder()
             .schema(Hello.class).build();

   // *** 2. Create the _hellos_ dataset. ***

   // Create the dataset using a URI, the inferred schema, and the supporting class.

      Dataset<Hello> hellos = Datasets.create(datasetUri, descriptor, Hello.class);

   // *** 3. Write a hello to the dataset. ***

   // Create a DatasetWriter instance.
      DatasetWriter<Hello> writer = null;

      try {

      // Create an instance of a Hello DatasetWriter.
         writer = hellos.newWriter();     

      // Create a Hello object instance.
         Hello hello = new Hello(person);

      // Write the Hello object to the dataset.
         writer.write(hello);

      } finally {

      // Ensure that the writer object closes.
         if (writer != null) writer.close();
      }
    
   // *** 4. Read the hello from the dataset ***

   // Create an instance of a Hello DatasetReader
      DatasetReader<Hello> reader = null;

      try {

      // Read the Hello objects from the dataset.
         reader = hellos.newReader();

      // Iterate through the Hello objects with the sayHello method.
         for (Hello hello : reader) {
            hello.sayHello();
         }

      } finally {

         if (reader != null) {

      // Ensure that the reader object closes.
         reader.close();
      }
   }
    
// *** 5. Delete the dataset now that you are done with it ***

   Datasets.delete(datasetUri);
   }
}
```

### HelloKite.class with ToolRunner

`Hello.class` defines the _hello_ object with its single field, _person_. `MakeDataset.class` performs Create, Add, Retrieve, and Delete functions on the `Hello` dataset. 

`HelloKite.class` implements a standard framework in which to execute the code via `ToolRunner`. This class implements the `main` and `run` methods. It extends `Configured.class`, which is used to instantiate configuration information, including the class path for compiling and running your application.

```Java
package org.kitesdk.examples.data;

import org.apache.hadoop.conf.Configured; // Provides configuration information.
import org.apache.hadoop.util.Tool;       // Supports handling of command-line options.
import org.apache.hadoop.util.ToolRunner; // Runs the application, modifies configuration.


public class HelloKite extends Configured implements Tool {

  @Override
  public int run(String[] args) throws Exception {
	MakeDataset datasetMaker = new MakeDataset();
	datasetMaker.makeDataset();
    return 0;
  }

  public static void main(String... args) throws Exception {
    int rc = ToolRunner.run(new HelloKite(), args);
    System.exit(rc);
  }
}
```

### Build and Run Hello Kite!

You can build the code for this example from the `dataset/` folder with this command:

```
mvn compile
```

You can run the example with this command:

```
mvn exec:java -Dexec.mainClass="org.kitesdk.examples.data.HelloKite"
```

`HelloKite` sends debug information to the console when it opens the reader. It writes a greeting to the console. Then it sends a debug message when it closes the reader.

Here is sample output from `HelloKite`.

```
2014-11-24 15:45:56 DEBUG :: Opening reader on path:file:/tmp/hellos/94c32428-eb38-4670-a9cf-6950cb4cb589.avro
Hello, Bob!
2014-11-24 15:45:56 DEBUG :: Closing reader on path:file:/tmp/hellos/94c32428-eb38-4670-a9cf-6950cb4cb589.avro
```

This example demonstrated the common tasks associated with creating and maintaining datasets. While, in practice, you are likely to create datasets of greater scope, these essential tasks remain the same, with variations to support your specific use case.
