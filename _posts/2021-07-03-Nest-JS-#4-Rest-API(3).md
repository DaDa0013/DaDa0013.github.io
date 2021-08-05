---
title: "Nest JS #4 Rest API(3)"
categories:
  - Nest JS
tags:
  - Nest JS
---



# 유효성 검사

## DTO

**DTO : 데이터 전송 객체(Data Transfer Object)**

-> 코드를 간결하게 하기 위해 사용

-> **NestJS 가 들어오는 query에 대해 유효성 검사**

* movies에 dto 폴더 생성

  

### 1.**create-movie.dto.ts 파일 생성**



* create-movie.dto.ts

  ```typescript
  export class CreateMovieDto{
      readonly title: string;
      readonly year: number;
      readonly genres: string[];
  }
  ```

* movies.controller 

  ```typescript
      @Post()
      create(@Body() movieData: CreateMovieDto){ //type은 dto
          return this.moviesService.create(movieData);
      }
  ```

* movies.service.ts

  ```typescript
      create(movieData: CreateMovieDto){
          this.movies.push({
              id: this.movies.length +1,
              ...movieData
          })
      }
  ```

* main.ts

  -> 유효성 검사용 파이프

  ```typescript
  import { ValidationPipe } from '@nestjs/common';
  import { NestFactory } from '@nestjs/core';
  import { AppModule } from './app.module';
  
  async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    app.useGlobalPipes(new ValidationPipe()) //ValidationPipe가 유효성 검사
    await app.listen(3000);
  }
  bootstrap();
  
  ```

* class-validator 와 class-transformer설치

  -> npm i class-validator class-transformer

* create-movie.dto.ts

  ```typescript
  import { IsNumber, IsString } from "class-validator";
  
  export class CreateMovieDto{
  
  @IsString()  //IsString 데코레이터
      readonly title: string;
  
  @IsNumber()
      readonly year: number;
      
  @IsString({each: true}) 
      readonly genres: string[];
  }
  ```

  

  ![image](https://user-images.githubusercontent.com/79195793/124348966-d4fb3400-dc27-11eb-8a40-ed19b1110f3a.png)



* **ValidationPipe**

  - 옵션

    - whitelist 

      -> 잘못된 정보는 Validator에 도달하지 X

    - forbidNonWhitelisted

      -> 리퀘스트 자체를 막기

      ![image](https://user-images.githubusercontent.com/79195793/124348977-db89ab80-dc27-11eb-9f61-09fb81da2ba3.png)

    * transform

      -> 입력데이터를 원하는 타입으로 바꿈

      => controller에서 수동으로 string을 number로 바꾸지 않아도 됨

  - main.ts

    ```typescript
    import { ValidationPipe } from '@nestjs/common';
    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';
    
    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
      app.useGlobalPipes(new ValidationPipe({
        whitelist:true,
        forbidNonWhitelisted:true,
        transform: true})) //ValidationPipe가 유효성 검사
      await app.listen(3000);
    }
    bootstrap();
    
    ```

  - movies.controller.ts

    ```typescript
    import { Body, Controller, Delete, Get, Param, Patch, Post, Put, Query } from '@nestjs/common';
    import { CreateMovieDto } from './dto/create-movie.dto';
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
        getOne(@Param("id") movieId: number): Movie{ //Movie 리턴
            console.log(typeof movieId);
            return this.moviesService.getOne(movieId);
        }
    
        @Post()
        create(@Body() movieData: CreateMovieDto){ //type은 dto
            return this.moviesService.create(movieData);
        }
    
        @Delete("/:id")
        remove(@Param('id') movieId: number){
            return this.moviesService.deleteOne(movieId);
        }
    
        @Patch('/:id')
        patch(@Param('id') movieId: number, @Body() updateData){
            return this.moviesService.update(movieId, updateData);
        }
    
    }
    
    ```

  - movies.service.ts

    ```typescript
    import { Injectable, NotFoundException } from '@nestjs/common';
    import { CreateMovieDto } from './dto/create-movie.dto';
    import { Movie } from './entities/movie.entity';
    
    @Injectable()
    export class MoviesService {
        private movies: Movie[]= [];
    
        getAll(): Movie[]{
            return this.movies;
        }
    
        getOne(id:number):Movie{
            const movie = this.movies.find(movie => movie.id === id); //+id는 string을 number로 변환 가능
            if(!movie){
                throw new NotFoundException(`Movie with ID ${id} not found.`) // 없는 경우 에러 메세지
            }
            return movie;
        }
    
        deleteOne(id:number){
            this.getOne(id); //id가 있는 경우 remove
            this.movies = this.movies.filter(movie => movie.id !== id);
        }
    
        create(movieData: CreateMovieDto){
            this.movies.push({
                id: this.movies.length +1,
                ...movieData
            })
        }
    
        update(id:number, updateData){
            const movie = this.getOne(id);
            this.deleteOne(id); //원래 id 지우기
            this.movies.push({...movie, ...updateData });
        }
    }
    
    ```

### 2. update-movie.dto.ts 파일 생성

* update-movie.dto.ts

  ```typescript
  import { IsNumber, IsString } from "class-validator";
  
  export class UpdateMovieDto{
  
  @IsString()  //IsString 데코레이터
      readonly title?: string; //읽기 전용 필수가 아니도록
  
  @IsNumber()
      readonly year?: number;
      
  @IsString({each: true}) 
      readonly genres?: string[];
  }
  ```

  또는

  1. npm i @nestjs/mapped-types      -> 타입을 변환시키고 사용할 수 있게 하는 패키지

  2. update-movie.dto.ts

     ```typescript
     import { PartialType } from "@nestjs/mapped-types";
     import { IsNumber, IsString } from "class-validator";
     import { CreateMovieDto } from "./create-movie.dto";
     
     export class UpdateMovieDto extends PartialType(CreateMovieDto){} //CreateMovieDto와 같지만 전부 필수사항은 아니게
     ```

     

* movies.controller.ts

  ```typescript
      @Patch('/:id')
      patch(@Param('id') movieId: number, @Body() updateData: UpdateMovieDto){
          return this.moviesService.update(movieId, updateData);
      }
  ```

* movies.service.ts

  ```typescript
      update(id:number, updateData: UpdateMovieDto){
          const movie = this.getOne(id);
          this.deleteOne(id); //원래 id 지우기
          this.movies.push({...movie, ...updateData });
      }
  ```

![image](https://user-images.githubusercontent.com/79195793/124348995-f3612f80-dc27-11eb-86e7-87fdc743b9d2.png)





## Modules and Dependency Injection

* **app.module 은 AppController와 AppProvider만 있어야 함**

  => **MoviesService와 MoviesController를 movies.module로 옮기기**(모듈 분리)

  

1. 모듈 생성

   * nest g m
   * 이름: movies

   => movies.module 생성

   

2. app.module에서 MovieService랑 Moviescontroller삭제

   ```typescript
   import { Module } from '@nestjs/common';
   import { MoviesModule } from './movies/movies.module';
   
   @Module({
     imports: [MoviesModule],
     controllers: [],
     providers: [],
   })
   export class AppModule {}
   
   ```

   

3. movies.modules.ts에 추가

   ```typescript
   import { Module } from '@nestjs/common';
   import { MoviesController } from './movies.controller';
   import { MoviesService } from './movies.service';
   
   @Module({
       controllers:[MoviesController],
       providers:[MoviesService]
   })
   export class MoviesModule {}
   
   ```

4. controller 생성

   ​	-> cli nest g co

   ​	->이름: app

   (controller는 src로 옮기기 )

   * app.controllers.ts

     ```typescript
     import { Controller, Get } from '@nestjs/common';
     
     @Controller('')
     export class AppController {
         @Get()
         home(){
             return 'Welcome to my Movie API'
         }
     }//홈페이지 가져오기
     
     ```
