---
id: CreateMessageBrokerClientOptionsFactory
title: Create Message Broker Client Options Factory
sidebar_label: Create Message Broker Client Options Factory
slug: /plugins/plugin-events/CreateMessageBrokerClientOptionsFactory
---


# Create Message Broker Client Options Factory

## Description

Creates the client options of the message broker.

This event does NOTHING by default. All of the logic is handled by the broker-kafka plugin https://github.com/amplication/plugins/tree/master/plugins/broker-kafka


## Event Name
`CreateMessageBrokerClientOptionsFactory`

## Event Params

There are no additional params to this event

```ts
export interface CreateMessageBrokerClientOptionsFactoryParams extends EventParams {}
```

Example:
```ts
  async afterCreateMessageBrokerClientOptionsFactory(
    context: DsgContext,
    eventParams: CreateMessageBrokerClientOptionsFactoryParams
  ): Promise<Module[]> {
    const { serverDirectories } = context;
    const filePath = resolve(staticDirectory, "generateKafkaClientOptions.ts");
    const file = await readFile(filePath, "utf8");
    const generateFileName = "generateKafkaClientOptions.ts";

    const path = join(
      serverDirectories.messageBrokerDirectory,
      generateFileName
    );

    return [
      {
        code: file,
        path,
      },
    ];
  }
```


