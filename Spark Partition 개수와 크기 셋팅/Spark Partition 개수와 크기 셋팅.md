.

Data_Engineering_TIL(20210101)

[참고자료]

‘Jaemun Jung’님 블로그글 “[Apache Spark] Partition 개수와 크기 정하기” 을 학습하고 정리한 노트입니다.

URL : https://jaemunbro.medium.com/apache-spark-partition-%EA%B0%9C%EC%88%98%EC%99%80-%ED%81%AC%EA%B8%B0-%EC%A0%95%ED%95%98%EA%B8%B0-3a790bd4675d

[학습내용]

![1](https://user-images.githubusercontent.com/41605276/103436028-c624cc00-4c5a-11eb-9237-ff51782378d7.jpg)

![2](https://user-images.githubusercontent.com/41605276/103436046-e8b6e500-4c5a-11eb-8f92-414abbe61b29.PNG)

![3](https://user-images.githubusercontent.com/41605276/103436332-9dea9c80-4c5d-11eb-8b46-96ddcd112050.jpg)

![4](https://user-images.githubusercontent.com/41605276/103436630-29b1f800-4c61-11eb-94ab-9c677714f0c1.jpg)

- 예시

![5](https://user-images.githubusercontent.com/41605276/103436642-5534e280-4c61-11eb-9587-4f43f4f55822.PNG)

![6](https://user-images.githubusercontent.com/41605276/103436903-2d934980-4c64-11eb-8f8c-f4ff8a586fda.jpg)

따라서

`파티션 개수 = 2850 / 0.9 = 3166` 가 된다.

위 공식들을 하나의 식으로 표현하면 아래와 같다.

![7](https://user-images.githubusercontent.com/41605276/103436922-5ddae800-4c64-11eb-881c-d9f7531bb581.PNG)

주의할 점은 위의 공식은 모든 레코드가 메모리로 읽어 들일 때 동일한 비율로 확장된다는 가정을 두고 있는데, 이 가정은 모든 상황에 항상 맞는 것은 아니다. 결론적으로 일반적인 케이스에서는 위의 공식을 참고하는데, 성능 향상이 더 이상 이루어지지 않을 때까지 파티션 개수를 늘려보는 것 이상의 확실한 대안은 없다.
