---
layout: post
title: 📖Django serialize in 4 minutes
date: 2024-05-13
categories: ["SamBeak", "Python", "Django", "Developer", "Backend"]
---

한바탕 이직을 하면서, <br>
관심 있던 Django에 대해 다시 공부해보았다. <br>
Django를 사용하면서 가장 많이 사용하는 것이 <br>
serialize이다. <br><br>

> # Django와 Python의 Serialize의 기초

<br>
시리얼라이제이션(serialization)은 객체를 <br>
특정 형식으로 변환하여 저장하거나<br>
네트워크를 통해 전송할 수 있게 하는 과정이다. <br><br>

deserialization은 그 반대의 과정을 의미한다. <br><br>

Django에서는 이러한 과정을 수행하는데 <br>
python의 내장 모듈인 pickle 또는 json을 사용해 <br>
데이터를 직렬화 할 수 있다 <br><br>

python의 pickle 모듈은 객체 그래프를 바이너리 형식으로 저장하며,<br>
대부분의 python 객체를 직렬화 할 수 있다. <br><br>

여기서 직렬화란 객체를 바이트 스트림으로 변환하는 것을 의미한다. <br>
그러나, 보안상의 이유로 신뢰할 수 없는 소스에서 받은 <br>
pickle 데이터를 로드하는 것은 위험하다. <br><br>

반면, json은 사람이 읽을 수 있는 텍스트 형식으로 데이터를 저장한다. <br>
딕셔너리, 리스트, 문자열, 숫자 등의 간단한 python 데이터 구조를 <br>
JSON 형식으로 직렬화 할 수 있다. <br><br>

Django에서는 models.py에 정의 된 모델 클래스의 <br>
인스턴스를 직렬화하여 데이터베이스에 저장하거나 <br>
HTTP requests/response 메시지에 포함시켜 전송할 수 있다. <br><br>

Django는 모델 클래스의 인스턴스를 직렬화 할 때 <br>
모델 클래스의 필드를 기반으로 직렬화를 수행한다. <br><br>

이 때, Django는 serialize 메서드를 사용해 <br>
모델 인스턴스를 사전 형태로 변환하고, <br>
deserialize 메서드를 사용해 사전을 모델 인스턴스로 변환한다. <br><br>

```python
from django.core import serializers

post = Post.objects.get(pk=1)
data = serializers.serialize('json', [post])
```

위 코드에서 'json'은 직렬화 형식을 의미하며, <br>
두 번째 인수는 직렬화 할 객체를 나타낸다. <br><br>

이렇게 직렬화 된 데이터는 일반적으로 파일 또는 데이터베이스에 저장된다. <br>
직렬화 된 데이터를 다시 모델 인스턴스로 변환하려면 <br>
즉, 역직렬화하려면 deserialize 메서드를 사용한다. <br><br>

> # 파이썬에서 데이터 시리얼라이즈 이해하기

<br>
시리얼라이제이션(serialization)은 복잡한 데이터 구조를 <br>
단순한 형식으로 변환하는 과정이다. <br>
이 과정을 통해 데이터를 저장하거나 <br>
네트워크를 통해 전송할 수 있게 된다. <br>
대표적인 예시가 바로 JSON이다. <br><br>

python에서는 JSON 모듈을 사용하여 <br>
시리얼라이즈 및 디시리얼라이즈를 수행할 수 있다. <br>
JSON 모듈은 Dictionary와 List를 JSON 형식으로 변환하거나 <br>
JSON 형식을 Dictionary와 List로 변환할 수 있다. <br><br>

```python
import json

data = {
    'name': 'Sam',
    'age': 25,
    'city': 'Seoul'
}
sam_data = json.dumps(data)
print(sam_data)

new_data = json.loads(sam_data)
print(new_data)
```

위 코드는 python의 Dictionary를 JSON 형식으로 변환하는 코드이다. <br>
dumps() 메서드는 Dictionary를 JSON 형식의 문자열로 변환한다. <br>
loads() 메서드는 JSON 형식의 문자열을 Dictionary로 변환한다. <br><br>

위 코드는 딕셔너리를 JSON 형식으로 변환하고, <br>
다시 JSON 형식을 딕셔너리로 변환하는 과정을 보여준다. <br><br>

객체 직렬화란 객체를 메모리 상의 상태에서 <br>
외부 형식으로 변환하는 과정이다. <br>
즉, 객체의 상태를 나타내는 값들을 모아서 <br>
외부 형식으로 표현하는 것이다. <br>
이러한 객체 직렬화는 주로 데이터베이스나 파일 시스템에 <br>
객체를 저장하거나 네트워크를 통해 객체를 전송할 때 사용된다. <br><br>

python에서는 pickle 모듈을 사용하여 객체를 직렬화 할 수 있다. <br>
그러나, 주의할 점은 pickle 모듈은 보안상의 이유로 <br>
악의적인 목적을 가진 사용자가 pickle 형식으로 인코딩 된 데이터를 <br>
수정하여 예상하지 못한 코드를 실행할 수 있다. <br><br>

따라서, pickle 모듈을 사용할 때는 신뢰할 수 있는 데이터만 <br>
pickle 형식으로 인코딩하고 디코딩해야 한다. <br><br>

> # Django 모델 인스턴스의 시리얼라이징 과정

<br>

장고에서는 모델의 필드 값들을 <br>
JSON 형식으로 변환하여 데이터베이스에 저장한다. <br>
이때, 각 필드는 해당 타입에 따라 적절한 방식으로 시리얼라이즈 된다. <br>
예를 들어, CharField는 문자열로, IntegerField는 정수로 변환된다. <br><br>
만약, 모델의 필드가 다른 모델을 참조하는 <br>
ForeignKey나 ManyToManyField인 경우, <br>
해당 모델의 기본 키 값이 JSON 형식으로 변환된다. <br><br>

이러한 Django의 시리얼라이징 과정 덕분에 <br>
개발자는 복잡한 데이터 처리를 직접하지 않아도 된다. <br>
하지만, 특정 필드를 직접 제어해야하는 경우는 <br>
serializer 클래스를 사용하여 직접 시리얼라이즈 할 수 있다. <br><br>
serializer 클래스는 ModelSerializer 클래스를 상속받아 <br>
필드 단위로 제어할 수 있는 메서드를 제공한다. <br>
예를 들어 필드를 숨기거나, 특정 필드만 시리얼라이즈 할 수 있다. <br>
이러한 과정은 API가 유연하게 동작하도록 하는데 도움이 된다. <br><br>
