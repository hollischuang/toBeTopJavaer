`List<?>` 是一个未知类型的List，而`List<Object>` 其实是任意类型的List。你可以把`List<String>`, L`ist<Integer>`赋值给`List<?>`，却不能把`List<String>`赋值给 `List<Object>`。
