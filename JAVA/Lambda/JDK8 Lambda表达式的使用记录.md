# JDK8 Lambda表达式的使用记录

## 1.逗号分隔

### （1）. List<对象>

```
List<Answer> answers = answerService.findByQuestionIdAndCorrect(questionId,correct);
String answerStr = answers.stream().map(Answer::getAnswerId).collect(Collectors.toList())
        .stream().map(w->w.toString()).collect(Collectors.joining(","));
```

### (2). List&lt;String&gt;

```
String result = list.stream().collect(Collectors.joining(""));
```

## 2.对象集合排序

```
List<Dict> dicts = dictService.findDict4Type(5);
// o1,o2 就是参数 代替匿名内部类写法
Collections.sort(dicts, ( o1, o2)->{
     Integer score1 = Integer.parseInt(o1.getDictKey());
     Integer score2 = Integer.parseInt(o2.getDictKey());
     return score1.compareTo(score2);
});
```



