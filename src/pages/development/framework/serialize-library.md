---
title: Serialize Library | Commerce PHP Extensions
description: Use this library in your Adobe Commerce and Magento Open Source components to manage the serialization and unserialization of data.
---

# Serialize library

This library provides a secure way of serializing and unserializing strings, integers, floats, booleans, and arrays.

Magento's Serialize library provides the `Magento\Framework\Serialize\SerializerInterface` and the Json and Serialize implementations for serializing data.

## Serialization

The main purpose of data serialization is to convert data into a string using `serialize()` to store in a database, a cache, or pass onto another layer in the application.

The other half of this process uses the `unserialize()` function to reverse the process and convert a serialized string back into string, integer, float, boolean, or array data.

<InlineAlert variant="warning" slots="text"/>

For security reasons, `SerializerInterface` implementations, such as the Json and Serialize classes, should not serialize and unserialize objects.

## Implementations

### Json (default)

The [`Magento\Framework\Serialize\Serializer\Json`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Serialize/Serializer/Json.php) class serializes and unserializes data using the [JSON](http://www.json.org/) format.

### JsonHexTag

The [`Magento\Framework\Serialize\Serializer\JsonHexTag`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Serialize/Serializer/JsonHexTag.php) class serializes and unserializes data using the [JSON](http://www.json.org/) format using the `JSON_HEX_TAG` option enabled.

### Base64Json

The [`Magento\Framework\Serialize\Serializer\Base64Json`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Serialize/Serializer/Base64Json.php) class serializes and encodes in the base64 format, and decodes the base64 encoded string and unserializes data using the [JSON](http://www.json.org/) format.

### Serialize

The [`Magento\Framework\Serialize\Serializer\Serialize`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Serialize/Serializer/Serialize.php) class is less secure than the Json implementation but provides better performance on large arrays.

### FormData

The [`Magento\Framework\Serialize\Serializer\FormData`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Serialize/Serializer/FormData.php) class unserializes the form data using the [JSON](http://www.json.org/) format. This class does not serialize objects to a form data format.

<InlineAlert variant="warning" slots="text"/>

Adobe discourages using the Serialize implementation directly because it can lead to security vulnerabilities. Always use the `SerializerInterface` for serializing and unserializing.

## Usage

Declare `SerializerInterface` as a [constructor dependency](../components/dependency-injection.md) to get an instance of a serializer class.

```php
use Magento\Framework\Serialize\SerializerInterface;

...

/**
 * @var SerializerInterface
 */
private $serializer;

...

public function __construct(SerializerInterface $serializer) {
  $this->serializer = $serializer;
}
```

\\
The following example shows how to use a serializer's `serialize()` and `unserialize()` functions to store and retrieve array data from a cache:

```php

...

/**
 * @var string
 */
private $cacheId = 'mySerializedData';

...

/**
 * Save data to cache
 * @param array $data
 *
 * @return bool
 */
public function saveDataToCache($data)
{
  return $this->getCache()->save($this->serializer->serialize($data), $this->cacheId);
}

...

/**
 * Load data from cache
 *
 * @return array
 */
public function loadDataFromCache()
{
  $data = $this->getCache()->load($this->cacheId);
  if (false !== $data) {
    $data = $this->serializer->unserialize($data);
  }
  return $data;
}
...
```

## Backward compatibility note

The `SerializerInterface` interface and its implementations only exist since Adobe Commerce and Magento Open Source version 2.2.
Because of this, it is not possible to use these classes in code that has to be compatible with Adobe Commerce and Magento Open Source 2.1 or 2.0.

In code that is compatible with earlier versions of Adobe Commerce and Magento Open Source, constructor dependency injection can not be used to get an instance of `SerializerInterface`.
Instead, a runtime check if the `SerializerInterface` definition exists can made, and if it does, it can be instantiated by directly accessing the object manager using a static method. Alternatively, a check against the Adobe Commerce and Magento Open Source version or the `magento/framework` Composer package version works too.
If the interface does not exist or an earlier version of Adobe Commerce and Magento Open Source is being executed, the appropriate native PHP serialization function has to be called, e.g. `\serialize()` or `\json_encode()`, depending on the usercase.

Here is an example:

```php
use Magento\Framework\Serialize\SerializerInterface;
use Magento\Framework\App\ObjectManager;

...
/**
 * @param mixed $data
 * @return string
 */
private function serialize($data)
{
    if (class_exists(SerializerInterface::class)) {
        $objectManager = ObjectManager::getInstance();
        $serializer = $objectManager->create(SerializerInterface::class);
        return $serializer->serialize($data);
    }
    return \serialize($data);
}
...

```
