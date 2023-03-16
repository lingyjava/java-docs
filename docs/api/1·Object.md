# 1·Object

- [1·Object](#1object)
  - [Object对象是什么](#object对象是什么)
  - [getClass()](#getclass)
  - [hashCode()](#hashcode)
  - [equals()](#equals)
  - [clone()](#clone)
  - [toString()](#tostring)
  - [notify()](#notify)
  - [notifyAll()](#notifyall)
  - [wait()](#wait)
  - [finalize()](#finalize)

## Object对象是什么
`java.lang.Object`

Object对象在Java中是所有对象的父类，其方法也比较重要。

## getClass()
`Class<?> getClass()`

获取类加载器。

## hashCode()
`int hashCode()`

获取hashCode值。

## equals()
`boolean equals(Object obj)`

比较两个对象。

## clone()
`Object clone()`

克隆对象。

## toString()
`String toString()`

对象转字符串。

## notify()
`void notify()`

线程方法，唤醒线程。

## notifyAll()
`void notifyAll()`

线程方法，唤醒所有线程。

## wait()
`void wait()`
`void wait(long timeout)`
`void wait(long timeout, int nanos)`

线程方法，使线程等待。

## finalize()
`void finalize()`
