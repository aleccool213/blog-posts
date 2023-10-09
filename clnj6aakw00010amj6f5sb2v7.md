---
title: "Typical vs. Protobufs: Data serialization in TypeScript"
datePublished: Fri Sep 15 2023 04:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clnj6aakw00010amj6f5sb2v7
slug: typical-vs-protobufs-data-serialization-in-typescript
canonical: https://blog.logrocket.com/typical-vs-protobuf-data-serialization-typescript/
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/8_NI1WTqCGY/upload/75344ce3e2bd48ce3f04378dab018203.jpeg
tags: typescript, kafka, protobuf, typical

---


Micro-services are prevalent in the software industry for their flexibility and scalability. Those micro-services will likely need to communicate with each other, not only that, clients like web apps and mobile apps also need to be sent information. One of the first problems software teams may encounter is making sure the right payloads are sent to and from services and clients. Developers either can assume payload structure through speaking to each other (docs, e-mail, etc) or they can opt into a technology which handles this for them through the use of shared schemas. Buying into one serialization technology makes it easy for developers share knowledge like schemas and build utilities for the common format which can lead to shipping code that is less prone to validation error. In a traditional REST API setting, OpenAPI is popular, but for other settings which use gRPC or messaging queues, technologies like [Typical](https://github.com/stepchowfun/typical), [Protobuf](https://protobuf.dev/), [Thrift](https://thrift.apache.org/) or [Avro](https://avro.apache.org/) are used. An example use-case could be two services within an organization which need to communicate employee data with each other over a topic in Kafka.

> Here is a [LogRocket article explaining Protobuf](https://blog.logrocket.com/using-protobuf-typescript-data-serialization/) in depth
> 

All of these technologies use something called a “schema”. This is used to serialize payloads into a binary and deserialize the binary into a payload. They offer type-safety in that the processes will fail if the payloads are not of the correct schema structure. Typical is based on functional principles, is relatively new to the scene with being around for two years and is maintained by a small set of core developers. In comparison, Protobuf has fifteen years behind it and is maintained by Google.  Both have a lot of similarities with the main differences accounting for the more modern features Typical has to offer. Typical has unique features like “asymmetric” fields for ensuring backward/forward compatibility and “choices” for supporting pattern-matching, while Protobuf support plugins and has a long-standing reputation and widespread use make it a more battle-tested solution for many.

To showcase the two technologies, we will go through some common tasks a developer will encounter when working with schemas when working with TypeScript. TypeScript is a very popular programming language which is a superset of JavaScript. We will go through a detailed tutorial working with employee data on how to serialize/deserialize data, how to safely make schema changes and what options there are for optional fields. 

## Our Example Employee service and Schemas

Lets pretend we have a service which sends data about employees to other services within an organization, lets call this service the “writer” (producer, serializer, publishers) when it comes to data interchanges and schemas. We will also handle receiving this employee data. These services can be called “readers” (consumers, deserializers, subscribers). We could use Kafka, GCP Pub Sub or a myriad of other technologies to send the data which supports binary transports. For the sake of examples and our tutorial, lets assume our services are written in TypeScript and are running on Node.js.

We will get our service setup to use both Typical and Protobuf so we can demonstrate the differences. The great thing about both of these technologies is that both offer a CLI tool to generate code based on a schema. It will generate serializers, deserializers and TypeScript types. This means we can use the schema as the source of truth for what payloads should look like, if changes need to be done, we can do it in the schema and run the generation over again. This can save you a lot of time and headache for developers who want to speed along coding. 

> A[ll of the code for this blog post can be referenced to this github repository](https://github.com/aleccool213/typical-vs-protobuf)
> 

## Setting up our development machine

To set everything up, we can run this script. We will install the Typical and Protobuf runtimes via [brew](https://brew.sh/). Ill be explicit how which steps are needed for each. The runtimes are needed for various functions like generating the types and code, etc. 

> It would be nice if these came packaged up on NPM but I couldn’t find packages suitable. Let me know if you find a better way.
> 

We will create a new folder for the project, create a `package.json` file with `npm init` and get TypeScript installed. 

```bash
mkdir typical-vs-protobuf-example & cd typical-vs-protobuf-example
brew install typical
brew install protobuf
// -- fix mac os js protobuf compiler issue
brew install protobuf@3
brew link --overwrite protobuf@3
// --
npm init -Y
npx tsc --init
npm add -D typescript ts-node @types/node
// Needed for TypeScript type gen in protobuf
npm add -D ts-protoc-gen
```

## The Schema

Here is a sample payload structure for the data we will be sending from service to service:

```json
{
    "id": "1",
    "name": "John Doe",
		"hourly_rate": 20,
    "department": "HR",
    "email": "john@doe.com",
    "active": true
}
```

The next step will be to define a schema for the payload, this defines what the payload should look like, if it doesn’t follow the schema, serialization and deserialization when running the service will fail.

> This is common functionality between most data interchange technologies. The schema is what provides “safety” to services and clients that they will receive the correct data in a shared format.
> 

In both Typical and Protobuf, the schemas will look similar with minor differences. We will define an `Employee` struct which contains basic information like `name` and `hourly_rate`, we will use an enum to offer a set of choices for `department`instead of a string as we only have two options it can be.

Typical:

```
// types.t

struct Employee {
    id: String = 1
    name: String = 2
    houry_rate: F64 = 3
    department: Department = 4
    email: String = 5
    active: Bool = 6
}

choice Department {
    HR = 0
    NOT_HR = 1
}
```

Protobuf:

```protobuf
// types.proto

syntax = "proto3";

package employee.v1;

message Employee {
    string id = 0;
    string name = 1;
    string email = 2;
    int32 hourly_rate = 3;
    Department department = 5;
    string email = 6;
    bool active = 7;

    enum Department {
        HR = 0;
        NOT_HR = 1;
    }
}
```

Once we have those defined, we can move onto the TypeScript code.

## Generating Serializers/Deserializers

Once we have the schema defined we can generate the TypeScript types and code using some NPM scripts. Add these scripts to your `package.json`:

```json
{
    "scripts" {
			"generate:typical:types:1": "typical generate typical-example/types-1.t --typescript typical-example/generated/types.ts",
	    "generate:protobuf:types:1": "protoc --plugin=\"protoc-gen-ts=./node_modules/.bin/protoc-gen-ts\" --ts_opt=esModuleInterop=true --js_out=\"./protobuf-example/generated\" --ts_out=\"./protobuf-example/generated\" ./protobuf-example/types-1.proto"
		}
}
```

Run the `generate:typical:types:1`  cmd and inspect the generated code in `typical-example/generated/types.ts. Do` the same for same for Protobuf but the path is, `protobuf-example/protobuf-example/typeos-1_pb.ts.`

Now we will write some code which uses it to serialize our sample payload from above into a binary. First in Typical: 

```tsx
import { Types1 } from "./generated/types";

// Take our sample payload
const payload = {
  id: "1",
  name: "John Doe",
  hourlyRate: BigInt(20),
  department: { $field: "hr" as const },
  email: "john@doe.com",
  active: true,
};

// Serialize the Employee object to binary using the generated Serializer from Typical
const binary = Types1.Employee.serialize(payload);

// Log that it was successful
console.log("Successfully serialized Employee object to binary:", binary);

// Send the binary off using Kafka, etc
...
```

Then in Protobuf:

```tsx
import { Employee } from "./protobuf-example/types-1_pb";

// Take our sample payload
const payload = {
  id: "1",
  name: "John Doe",
  hourlyRate: 20,
  department: Employee.Department.HR,
  email: "john@doe.com",
  active: true,
};

// Create the Employee object based on our sample payload
const employee = new Employee();
employee.setId(payload.id);
employee.setName(payload.name);
employee.setHourlyRate(payload.hourlyRate);
employee.setDepartment(payload.department);
employee.setEmail(payload.email);
employee.setActive(payload.active);

// Serialize the Employee object to binary
const binary = employee.serializeBinary();

// Log that it was successful
console.log("Successfully serialized Employee object to binary:", binary);

// Send the binary off using Kafka, etc
...
```

As you can see, there is still is not many differences. When it comes to schema changes and optionals is where things start to change.

## Making a schema change or schema evolution

We realized we made a crucial error, we need to add an employee’s phone number to the schema. All employees have to input the phone number into the service so it will always be present. After some thought we decide to make this field required. Depending on the technology used and the quantity of readers and writers, it may be a breaking schema change. This means that we need to be careful to not send incompatible payloads to services which can’t handle them.

For example, let’s say we have multiple writers using this schema, if we add a required field, some writers will not be updated at the same time (we are part of a big organization). If this is the case readers can’t expect it to be present on every single message until every writer has there code and schema updated.

> This is a big topic when it comes to messaging systems so we will only focus on the differences between Typical and Protobuf to go about this kind of change.
> 

The question is how do we roll this change our safely and effectively. There are other changes we may want to make.

Some definitions:

- Backwards Compatible
    - Means writers send messages of a new schema version and readers can still process messages using an old schema version.
- Forwards Compatible
    - Means readers updated to a new schema version can still process messages with writers using an old schema version.

First lets dig into how Typical does it. Every change is forwards and backwards compatible which makes things easy to understand. There is a feature called “asymmetric” fields which were made for this use-case. 

> There are no non-nullable fields in Typical, which is a big difference compared to other technologies.
> 

How this works if that you add the keyword on a field. This basically says that its required for the writer, optional for the reader. When we are 100% sure all of the writers have been updated, we remove the keyword which makes the field required. All fields in Typical are required by default.

Lets see an example schema:

```
struct Employee {
    id: String = 1
    ...
    asymmetric phone_number: String = 7
}
```

Now that we have the schema, run type generation again and inspect the outputted types:

```tsx
export type EmployeeOut = {
  id: string;
  ...
  phoneNumber: string;
};

export type EmployeeIn = {
  id: string;
  ...
  phoneNumber?: string;
};
```

You can see how it is now optional for the readers. Here is a [summary of schema changes which are safe](https://github.com/stepchowfun/typical#summary-of-what-kinds-of-schema-changes-are-safe) in Typical.

Now with Protobuf, all fields are required by default. But the main difference is that there is a traditional `optional` keyword which you can use. This makes the rules on how you can evolve a schema a more involved. Some changes are forwards compatible and some are backwards, overall its more granular approach.

> [Here is a good article which summarizes what kind of changes are forwards/backwards compatible](https://softwaremill.com/schema-evolution-protobuf-scalapb-fs2grpc/)
> 

Now our strategy must goes as follows:

1. Update the Protobuf schema to have `phoneNumber` as optional
2. Update all writers to use the new schema and update the code to have the value be present
3. Update all readers with the new schema
4. Wait until all writers have finished updating
5. Update the Protobuf schema to have `phoneNumber` as required
6. Repeat steps 2, 3, and 4

You can see it’s more involved and you have to know what you are doing. This process is similar to Thrift and Avro.

## Flexible Payloads

You may have payloads where optional fields make sense. An example could be filling the success and error fields inside of a payload. When a success message is sent, error would not be present and vice verse.

In Typical, the spec doesn’t support non-nullable fields but instead uses something called “choices”. This offers pattern-matching-like capabilities to readers and acts like an enum (we used it before for `department`). It’s much more flexible than a traditional enum as fields inside of a choice can be strings or new structs.

Here is an example of adding a `details` field to the Employee struct which explains to a reader if it was successful or an error occurred in the payload.

Here is the Typical schema:

```
struct Employee {
    id: String = 1
    ...

    details: Details = 7
}

choice Details {
    success = 0
    error: String = 1
}
```

Here is what the writer may serialize:

```tsx
const payload: Types2.EmployeeOut = {
  id: "1",
  ...
  details: {
    $field: "success",
  },
};
```

Here is what the reader may deserialize:

```tsx
// Read the binary payload from a file
const fileContents = readFileSync(filePath);
// Deserialize using Typical generated code
const payloadDeserialized = Types2.Employee.deserialize(
  new DataView(
    fileContents.buffer,
    fileContents.byteOffset,
    fileContents.byteLength
  )
);
// Handle the details field
switch (payloadDeserialized.details.$field) {
  case "success":
    console.log("We have a success!");
    break;
  case "error":
    console.log("We have an error!");
    break;
  default:
    throw new Error("Unknown details field");
}

```

In Protobuf you can use the `optional` keyword like we described before. If you want to mimic the behaviour of a choice in Typical, [there is a keyword `oneof`](https://protobuf.dev/programming-guides/proto3/#oneof). This can codify different options a field can take and you can define more than a traditional enum.

Here is an example of using the same schema as above using a `oneof` :

```protobuf

message Employee {
    string id = 1;
    ...

    oneof details {
        bool success = 7;
        string error = 8;
    }
}
```

Here is what the reader may deserialize:

```tsx
...
// Read the binary payload from a file
const fileContents = readFileSync(filePath);
const payloadDeserialized = Employee.deserializeBinary(fileContents);

// Handle the details field
switch (payloadDeserialized.getDetailsCase()) {
  case Employee.DetailsCase.SUCCESS:
    console.log("We have a success!");
    break;
  case Employee.DetailsCase.ERROR:
    console.log("We have an error!");
    break;
  default:
    throw new Error("Unknown details field");
}
```

## Conclusion

While Typical offers some unique features like asymmetric fields for ensuring backward/forward compatibility and choices for pattern-matching, Protobuf's long-standing reputation and widespread also makes it an attractive choice. Being a relatively new technology, Typical can offer some advantages like a single cohesive CLI tool which can generate types and code instead of having to download separate packages in Protobuf. On the other hand, Protobuf supports plugins and has a wider range of language support compared to Typical. It also offers traditional optional fields, making schema evolution more involved but also more granular when compared to the opinionated approach of Typical.

In conclusion, both Typical and Protobuf have their own unique advantages and limitations. The choice between the two depends on the specific needs and preferences of the organization.