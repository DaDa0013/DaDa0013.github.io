---
title: "Nest JS #3 Rest API(2)"
categories:
  - Nest JS
tags:
  - Nest JS
toc: true
toc_sticky: true
---

# Single-responsibility principle

: 하나의 module, class 혹은 function이 하나의 기능을 꼭 책임져야함



# Service 만들기

* 서비스-> 로직을 관리

* 컨트롤러: url 매핑, 리퀘스트 받기, Query 넘기기, Body나 그 외 넘기기

  

1. terminal에 nest g s   // 이름: movies

   **=> movies.service.ts 생김**

   **=> app.module에 자동으로 providers에 moviesservice 추가됨**

   

2. **movies.service에 데이터베이스 만들기**

   (1) movies에 entities 폴더 생성 -> movie.entity.ts 파일 생성

    **=> entities에 데이터베이스 모델 만들기**

   * movie.entity.ts

     ```typescript
     export class Movie{
         id: number;
         title: string;
         year: number;
         genres: string[];
     }
     ```

   * app.service.ts

     ```typescript
     import { Injectable } from '@nestjs/common';
     import { Movie } from './entities/movie.entity';
     
     @Injectable()
     export class MoviesService {
         private movies: Movie[]= [];
     
         getAll(): Movie[]{
             return this.movies;
         }
     
         getOne(id:string):Movie{
             return this.movies.find(movie => movie.id === +id); //string을 number로 변환 가능
         }
     
         deleteOne(id:string):boolean{
             this.movies.filter(movie => movie.id !== +id);
             return true;
         }
     
         create(movieData){
             this.movies.push({
                 id: this.movies.length +1,
                 ...movieData
             })
         }
     }
     
     ```
     
     -> 위는 가짜 데이터베이스
     
     -> **진짜 데이터베이스는 데이터베이스에 대한 Query가 옴**

   ** function은 똑같은 이름해도 상관 없음**

   * movies.controller.ts

     ```typescript
     import { Body, Controller, Delete, Get, Param, Patch, Post, Put, Query } from '@nestjs/common';
     import { Movie } from './entities/movie.entity';
     import { MoviesService } from './movies.service';
     
     @Controller('movies')
     export class MoviesController {
     
         constructor(private readonly moviesService: MoviesService){}//읽기 전용 movieService 클래스를 가짐
     
         @Get()
         getAll(): Movie[]{ //Movie array 리턴
             return this.moviesService.getAll(); //consturctor 덕분에 쓸 수 있음
         }
     
         @Get("/:id")
         getOne(@Param("id") movieId: string): Movie{ //Movie 리턴
             return this.moviesService.getOne(movieId);
         }
     
         @Post()
         create(@Body() movieData){
             return this.moviesService.create(movieData);
         }
     
         @Delete("/:id")
         remove(@Param('id')movieId:string){
             return this.moviesService.deleteOne(movieId);
         }
     
         @Patch('/:id')
         patch(@Param('id') movieId: string, @Body() updateData){
             return {
                 updatedMovie:movieId,
                 ...updateData,
             };
         }
     }
     ```

   * 실행

     1. getAll

        ![image](https://user-images.githubusercontent.com/79195793/124347682-d70dc480-dc20-11eb-94e5-f540db9b63c4.png)

        -> 빈 array 실행

     2. post

        ![image](https://user-images.githubusercontent.com/79195793/124347695-e12fc300-dc20-11eb-8d6b-a7522f981c99.png)

     3. getOne

        ![image](https://user-images.githubusercontent.com/79195793/124347701-eab92b00-dc20-11eb-8b2a-94f5cf3b817e.png)

# service

-> 파일을 저장하면 데이터베이스 전부가 날라감

​	(메모리 데이터 베이스이기 때문에 서버를 켜놓는 동안만 데이터베이스가 기억)



* **없는 id 경우 에러화면 내보내기**

  * movies.controllers.ts

    ```typescript
        @Patch('/:id')
        patch(@Param('id') movieId: string, @Body() updateData){
            return this.moviesService.update(movieId, updateData);
        }  
    ```

    

  * movies.servies.ts

    ```typescript
        getOne(id:string):Movie{
            const movie = this.movies.find(movie => movie.id === +id); //string을 number로 변환 가능
            if(!movie){
                throw new NotFoundException(`Movie with ID ${id} not found.`) // 없는 경우 에러 메세지
            }
            return movie;
        }
    
    
        deleteOne(id:string){
            this.getOne(id); //id가 있는 경우 remove
            this.movies = this.movies.filter(movie => movie.id !== +id);
        }
    
    
        update(id:string, updateData){
            const movie = this.getOne(id);
            this.deleteOne(id); //원래 id 지우기
            this.movies.push({...movie, ...updateData });
        }
    ```

    

    

    * getOne

    ![image](https://user-images.githubusercontent.com/79195793/124347714-f573c000-dc20-11eb-9dc1-ef301ffaaeff.png)

    * delete

    ![image](https://user-images.githubusercontent.com/79195793/124347723-fd336480-dc20-11eb-8b95-fc46843c2ba4.png)

    * patch

    ![image](https://user-images.githubusercontent.com/79195793/124347732-03c1dc00-dc21-11eb-8343-9f8bc8f5e9b9.png)

    ![image](https://user-images.githubusercontent.com/79195793/124347742-0ae8ea00-dc21-11eb-96ce-1d3b4c9d0b3e.png)

    ![image](https://user-images.githubusercontent.com/79195793/124347746-120ff800-dc21-11eb-9466-0985affedfda.png)

