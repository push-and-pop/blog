---
title: golang timer的坑
date: 2024-12-31 14:16:25
tags: golang
hidden: true
---

```golang
package main

import (
	"fmt"
	"runtime"
	"time"
)

type Manager struct {
	ptrTimer *time.Timer
}

func main() {
	fmt.Println("Main Begin Time:", time.Now().Format("15:04:05"))

	manager := &Manager{}
	runtime.SetFinalizer(manager, func(data *Manager) {
		fmt.Printf("runtime invoke Finalizer manager, time: %s\n", time.Now().Format("15:04:05"))
	})

	LifeFunc(manager)
	manager = nil

	for {
		runtime.GC()
		time.Sleep(1 * time.Second)
	}
}

func LifeFunc(manager *Manager) {
	manager.ptrTimer = DoWithDebug(manager)
	// manager.ptrTimer.Stop()
	// manager.ptrTimer = nil
}

func DoWithDebug(manager *Manager) *time.Timer {
	timer := time.AfterFunc(30*time.Second, func() {
		_ = manager // manager被闭包引用
		fmt.Printf("timer invoke : time: %s\n", time.Now().Format("15:04:05"))
		// ...logic
	})

	runtime.SetFinalizer(timer, func(data *time.Timer) {
		fmt.Printf("runtime invoke Finalizer timer: time: %s\n", time.Now().Format("15:04:05"))
	})

	return timer
}

```



# 总结

本质是相互引用导致的内存泄漏，但是timer执行完或者stop也没有释放闭包里的引用也算是一个坑
