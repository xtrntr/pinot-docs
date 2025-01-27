---
description: >-
  This section contains a collection of guides that will show you how to import
  data from a Pinot supported input format.
---

# Input formats

Pinot offers support for various popular input formats during ingestion. By changing the input format, you can reduce the time that goes in serialization-deserialization and speed up the ingestion.

The input format can be changed using the `recordReaderSpec` config in the ingestion job spec.

```text
recordReaderSpec:
  dataFormat: 'csv'
  className: 'org.apache.pinot.plugin.inputformat.csv.CSVRecordReader'
  configClassName: 'org.apache.pinot.plugin.inputformat.csv.CSVRecordReaderConfig'
  configs: 
			key1 : 'value1'
			key2 : 'value2'
```

The config consists of the following keys -

`dataFormat` - Name of the data format to consume.

`className` - name of the class that implements the `RecordReader` interface. This class is used for parsing the data.

`configClassName` - name of the class that implements the `RecordReaderConfig` interface. This class is used the parse the values mentioned in `configs`

`configs` - Key value pair for format specific configs. This field can be left out.

Pinot supports the multiple input formats out of the box. You just need to specify the corresponding readers and the associated custom configs to switch between the formats.

#### CSV

```text
dataFormat: 'csv'
className: 'org.apache.pinot.plugin.inputformat.csv.CSVRecordReader'
configClassName: 'org.apache.pinot.plugin.inputformat.csv.CSVRecordReaderConfig'
configs:
	fileFormat: 'default' #should be one of default, rfc4180, excel, tdf, mysql
	header: 'columnName seperated by delimiter'
  delimiter: ','
  multiValueDelimiter: '-'
```

CSV Record Reader supports the following configs -

`fileFormat` - can be one of default, rfc4180, excel, tdf, mysql

`header` - header of the file. The columnNames should be seperated by the delimiter mentioned in the config

`delimiter` - The character seperating the columns

`multiValueDelimiter` - The character seperating multiple values in a single column. This can be used to split a column into a list.

{% hint style="info" %}
Your CSV file may have raw text fields that cannot be reliably delimited using any character. In this case, explicitly set the **multiValueDelimeter** field to empty in the ingestion config.   
  
`multiValueDelimiter: ''` 
{% endhint %}

#### AVRO

```text
dataFormat: 'avro'
className: 'org.apache.pinot.plugin.inputformat.avro.AvroRecordReader'
```

The Avro record reader converts the data in file to a `GenericRecord`. A java class or `.avro` file is not required.

#### JSON

```text
dataFormat: 'json'
className: 'org.apache.pinot.plugin.inputformat.json.JSONRecordReader'
```

#### Thrift

```text
dataFormat: 'thrift'
className: 'org.apache.pinot.plugin.inputformat.thrift.ThriftRecordReader'
configs:
	thriftClass: 'ParserClassName'
```

**Note**: Thrift requires the generated class using `.thrift` file to parse the data. The .class file should be available in the Pinot's classpath. You can put the files in the `lib/` folder of pinot distribution directory.

#### Parquet

```text
dataFormat: 'parquet'
className: 'org.apache.pinot.plugin.inputformat.parquet.ParquetRecordReader'
```

{% hint style="warning" %}
The above class doesn't read the Parquet `INT96` and `Decimal`type.
{% endhint %}

Please use the below class to handle  `INT96` and `Decimal`type.

```text
dataFormat: 'parquet'
className: 'org.apache.pinot.plugin.inputformat.parquet.ParquetNativeRecordReader'
```

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parquet Data Type</th>
      <th style="text-align:left">Java Data Type</th>
      <th style="text-align:left">Comment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">INT96</td>
      <td style="text-align:left">INT64</td>
      <td style="text-align:left">
        <p>Parquet<code>INT96</code> type converts <b>nanoseconds</b>
        </p>
        <p>to Pinot <code>INT64</code> type of <b>milliseconds</b>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">DECIMAL</td>
      <td style="text-align:left">DOUBLE</td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>

#### ORC

```text
dataFormat: 'orc'
className: 'org.apache.pinot.plugin.inputformat.orc.ORCRecordReader'
```

ORC record reader supports the following data types -

| ORC Data Type | Java Data Type |
| :--- | :--- |
| BOOLEAN | String |
| SHORT | Integer |
| INT | Integer |
| LONG | Integer |
| FLOAT | Float |
| DOUBLE | Double |
| STRING | String |
| VARCHAR | String |
| CHAR | String |
| LIST | Object\[\] |
| MAP | Map&lt;Object, Object&gt; |
| DATE | Long |
| TIMESTAMP | Long |
| BINARY | byte\[\] |
| BYTE | Integer |

{% hint style="info" %}
In LIST and MAP types, the object should only belong to one of the data types supported by Pinot.
{% endhint %}

#### Protocol Buffers

```text
dataFormat: 'proto'
className: 'org.apache.pinot.plugin.inputformat.protobuf.ProtoBufRecordReader'
configs:
	descriptorFile: 'file:///path/to/sample.desc'
```

The reader requires a descriptor file to deserialize the data present in the files. You can generate the descriptor file \(`.desc`\) from the `.proto` file using the command -

```text
protoc --include_imports --descriptor_set_out=/absolute/path/to/output.desc /absolute/path/to/input.proto
```

