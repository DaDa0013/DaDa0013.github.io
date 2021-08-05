---
title: "우버이츠클론코딩 #2 GRAPHQL API"
categories:
  - clone coding
  - Uber Eats
  - Nest JS
  - typeORM
tags:
  - clone coding
  - Uber Eats
  - Nest JS
  - typeORM
---

# GRAPHQL

1. graphql 설치

   * npm i @nestjs/graphql graphql-tools graphql apollo-server-express

     -> Nestjs 방식으로, apollo-server-express 기반으로 작동

2. controller랑 service 다 지우기

   app.module : main.ts로 import 되는 유일한 모듈

   ```typescript
   import { Module } from '@nestjs/common';
   import { GraphQLModule } from '@nestjs/graphql';
   
   @Module({
     imports: [
       GraphQLModule.forRoot()//root module 설정
     ],
     controllers: [],
     providers: [],
   })
   export class AppModule {}
   
   ```

   -> graphql 은 shema와 resolver 필요

   

3. restaurant 모듈( 개념이해를 위해 잠시 만들기)

    * nest g mo restaurants

      * restaurants에 restaurants.resolvers.ts 파일 생성

      * restaurants.resolvers.ts

        ```typescript
        import { Query, Resolver } from "@nestjs/graphql";
        
        
        @Resolver()
        export class RestaurantResolver {
            @Query(() => Boolean) //graphql에 return
            isPizzaGood(): Boolean{ //typescript에 return
                return true;
            }
        }
        
        ```

      => localhost:3000/graphql

   * restaurants에 entities폴더 생성

   * restaurant.entity.ts 생성

     - entities: 데이터베이스에 있는 모델

     - restaurant.entity.ts

       ```typescript
       import { Field, ObjectType } from "@nestjs/graphql";
       
       
       @ObjectType()
       export class Restaurant{//Restaurant을 위한 object type 생성
           @Field(is => String)
           name: string;
           @Field(type => Boolean,{nullable:true})
           isGood: boolean;
       }
       ```

       

     - restaurant.resolvers.ts

       ```typescript
       import { Query, Resolver } from "@nestjs/graphql";
       import { Restaurant } from "./entities/restaurant.entity";
       
       
       @Resolver(of => Restaurant)
       export class RestaurantResolver {
           @Query(returns => Restaurant) //리턴 restaurant
           myRestaurant(){
               return true; //restaurant를 리턴해야하는데 true를 리턴하므로 에러
           }
       }
       ```

       

       ![image](https://user-images.githubusercontent.com/79195793/125108647-49990b80-e11d-11eb-842e-ebcd2be944a4.png)

## Mutation

* restaurants.resolver.ts

  * arguments 만들기

  ```typescript
  
  import { Args, Query, Resolver } from "@nestjs/graphql";
  import { Restaurant } from "./entities/restaurant.entity";
  
  
  @Resolver(of => Restaurant)
  export class RestaurantResolver {
      @Query(returns => [Restaurant]) //리턴 restaurant array
      restaurants(@Args('veganOnly') veganOnly: boolean): Restaurant[] { //veganOnly라는 argument 호출
          return [];   
      }
  }
  ```

  ![image](https://user-images.githubusercontent.com/79195793/125108690-57e72780-e11d-11eb-9733-9884129d18f4.png)

* restaurants.resolvers.ts

  * mutaiton

    ```typescript
    
    import { Args, Mutation, Query, Resolver } from "@nestjs/graphql";
    
    import { Restaurant } from "./entities/restaurant.entity";
    
    
    @Resolver(of => Restaurant)
    export class RestaurantResolver {
        @Query(returns => [Restaurant]) //리턴 restaurant array
        restaurants(@Args('veganOnly') veganOnly: boolean): Restaurant[] { //veganOnly라는 argument 호출
            console.log(veganOnly)
            return [];   
        }
    
        @Mutation(returns => Boolean)
        createRestaurant(): boolean {
            return true;
        }
    ```

    ![image](https://user-images.githubusercontent.com/79195793/125108716-60d7f900-e11d-11eb-82c9-9e1f367416a8.png)

* restaurant.entity.ts

  ```typescript
  import { Field, ObjectType } from "@nestjs/graphql";
  
  
  @ObjectType()
  export class Restaurant{//Restaurant을 위한 object type 생성
      @Field(is => String)
      name: string;
  
      @Field(type => Boolean)
      isVegan: boolean;
  
      @Field(type => String)
      address:string;
  
      @Field(type => String)
      ownerName:string;
  }
  ```

* restaurants.resolver.ts

  ```typescript
  
  import { Args, Mutation, Query, Resolver } from "@nestjs/graphql";
  
  import { Restaurant } from "./entities/restaurant.entity";
  
  
  @Resolver(of => Restaurant)
  export class RestaurantResolver {
      @Query(returns => [Restaurant]) //리턴 restaurant array
      restaurants(@Args('veganOnly') veganOnly: boolean): Restaurant[] { //veganOnly라는 argument 호출
          console.log(veganOnly)
          return [];   
      }
  
      @Mutation(returns => Boolean)
      createRestaurant( //arguments 만들기
          @Args('name') name:string,
          @Args('isVegan') isVegan:boolean,
          @Args('address') address:string,
          @Args('ownerName') ownerName:string,
      ): boolean{
          return true;
      }
  }
  ```

  ![image](https://user-images.githubusercontent.com/79195793/125108741-69303400-e11d-11eb-8993-1030811a075e.png)

  => 위처럼 argument 하나씩 하는 것보다 InputType으로 한번에 object를 전달하게 하는게 편함(dto랑 비슷)

  ## InputType

  restaurants에 dtos 폴더 생성 -> create-restaurant.dto.ts 파일 생성

* create-restaurant.dto.ts

  ```typescript
  import { Field, InputType } from "@nestjs/graphql";
  
  @InputType()
  export class CreateRestaurantDto {
      @Field(type => String) //Field는 Type을 필요로 함
      name: string;
      @Field(type => Boolean)
      isVegan: boolean;
      @Field(type => String)
      address: string;
      @Field(type => String) 
      ownerName: string;
  }
  ```

* restaurants.resolvers.ts

  ```typescript
      @Mutation(returns => Boolean)
      createRestaurant( //arguments 만들기
          @Args('createRestaurantInput') createRestaurantInput: CreateRestaurantDto //한번에 묶기
      ): boolean{
          console.log(createRestaurantInput)
          return true;
      }
  }
  ```

  ![image](https://user-images.githubusercontent.com/79195793/125108760-70efd880-e11d-11eb-8e67-9bf882150910.png)

![image](https://user-images.githubusercontent.com/79195793/125108783-76e5b980-e11d-11eb-9874-c1d8033bb645.png)
​	-> 위와 같이 써야하는데 불편함

=> **InputType을 ArgsType으로 바꾸기**

	## 	ArgsType

* create-restaurants.dto.ts

  ```typescript
  import { ArgsType, Field, InputType } from "@nestjs/graphql";
  
  @ArgsType() //각각을 분리된 arguments로써 정의 & 분리된 값을 GraphQL argument로 전달
  export class CreateRestaurantDto {
      @Field(type => String) //Field는 Type을 필요로 함
      name: string;
      @Field(type => Boolean)
      isVegan: boolean;
      @Field(type => String)
      address: string;
      @Field(type => String) 
      ownerName: string;
  }
  ```

* restaurants.resolver.ts

  ```typescript
      @Mutation(returns => Boolean)
      createRestaurant( //arguments 만들기
          @Args() createRestaurantDto: CreateRestaurantDto
      ): boolean{
          console.log(createRestaurantDto)
          return true;
      }
  }
  ```

  

![image](https://user-images.githubusercontent.com/79195793/125108805-7ea55e00-e11d-11eb-91c1-f7b321e016d6.png)


## dto class validators

npm i class-validator

npm i clas-transformer

* create-restaurant.dto.ts

  ```typescript
  import { ArgsType, Field, InputType } from "@nestjs/graphql";
  import { IsBoolean, IsString, Length } from "class-validator";
  
  @ArgsType() //각각을 분리된 arguments로써 정의 & 분리된 값을 GraphQL argument로 전달
  export class CreateRestaurantDto {
  
      @Field(type => String) //Field는 Type을 필요로 함
      @IsString()
      @Length(5, 10)
      name: string;
      
      @Field(type => Boolean)
      @IsBoolean()
      isVegan: boolean;
  
      @Field(type => String)
      @IsString()
      address: string;
  
      @Field(type => String) 
      @IsString()
      @Length(5, 10) //최소 최대 길이 제한
      ownerName: string;
  }
  ```

* main.ts

  ```typescript
  import { ValidationPipe } from '@nestjs/common';
  import { NestFactory } from '@nestjs/core';
  import { AppModule } from './app.module';
  
  async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    app.useGlobalPipes(
      new ValidationPipe() 
    )
    await app.listen(3000);
  }
  bootstrap();
  ```

  ![image](https://user-images.githubusercontent.com/79195793/125108845-882ec600-e11d-11eb-942d-5bd640071d20.png)

  길이 검사해줌
