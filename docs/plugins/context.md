---
id: context-skip-default
title: Context and Skip Default Behavior
sidebar_label: Context and Skip Default Behavior
slug: /plugins/context-skip-default
---

## Context

In each lifecycle function (before and after), we have access to a global object called **context** which holds common data between event

| Name                | Type                          | Description                                                                                                                                                                                          |
| ------------------- | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| resourceType        | keyOf typeOf EnumResourceType | the type of the resource: `MessageBroker` `ProjectConfiguration` `Service`                                                                                                                           |
| resourceInfo        | AppInfo                       | the general data of the app:name, description, version, id, URL, and settings (resource settings)                                                                                                    |
| entities            | Entity[]                      | the list of entities in the service                                                                                                                                                                  |
| roles               | roles                         | the list of roles in the service                                                                                                                                                                     |
| pluginInstallations | pluginInstallations           | a list of the installed plugins                                                                                                                                                                      |
| topics              | Topic[]                       | a list of topics that are connected to a specific service                                                                                                                                            |
| modules             | ModuleMap                      | Map of the generated modules (files)                                                                                                                                                                |
| DTOs                | DTOs                          | list of the generated DTOs                                                                                                                                                                           |
| plugins             | PluginMap                     | a `Map` of the event name with its before and after function, and event params                                                                                                                       |
| logger              | BuildLogger                   | a logger that can be used to log messages inside the plugin. It will allow to create logs in the user facing build log                                                                               |
| utils               | ContextUtil                   | an interface that exposes utility functionalities and properties, such as `skipDefaultBehavior` - will be explained later, and `importStaticModules` - provides access to the static generated files |
| clientDirectories   | clientDirectories             | helps to determine where a specific file or folder will be generated in the generated admin-ui folder ` baseDirectory``srcDirectory``authDirectory``publicDirectory``apiDirectory `                  |
| serverDirectories   | serverDirectories             | helps to determine where a specific file or folder will be generated in the generated server folder `baseDirectory` `srcDirectory` `authDirectory` `scriptsDirectory` `messageBrokerDirectory`       |

## Skip Default Behavior

In the `before` function, you can call the `skipDefaultBehavior` from the context and set it to true (by default its value is false):

```tsx
context.utils.skipDefaultBehavior = true;
```

As a result, the default functionality won’t be executed and you will have to provide your own logic. You can do it after the skip in the `before` function, or in the `after` function.

:::important

Using `skipDefaultBehavior` and not providing an alternative functionality can cause unexpected behavior. For example, if the skipped event generates files that are imported from other files, so use it wisely and carefully.

:::
