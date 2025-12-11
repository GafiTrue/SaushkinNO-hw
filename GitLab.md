# Домашнее задание к занятию "`GitLab`" - `Саушкин Николай Олегович`

---

### Задание 1
Развернуть GitLab и настроить Runner

![photo_2025-12-12_01-08-05](https://github.com/user-attachments/assets/a03c1171-0bb1-471f-ab78-48ef899416d0)
![photo_2025-12-12_01-09-21](https://github.com/user-attachments/assets/6bb57f51-b902-4353-9965-816b5a2a4715)

---

### Задание 2
Задание 2: Настройка CI/CD Pipeline

Файл .gitlab-ci.yml
```
stages:
  - test
  - build

test:
  stage: test
  image: golang:1.17
  tags:
    - docker
  script:
    - go test .

build:
  stage: build
  image: docker:latest
  tags:
    - docker
  only:
    - main
    - master
  script:
    - docker build .
```
![photo_2025-12-12_01-03-49](https://github.com/user-attachments/assets/03e08f46-82d7-4985-88f4-134c864230f0)

![photo_2025-12-12_01-15-45](https://github.com/user-attachments/assets/5ff7044e-abb2-4b1b-920f-9b6426958781)



