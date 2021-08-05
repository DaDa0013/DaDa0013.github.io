---
title: "NestJS #5 UNIT TESTING"
categories:
  - Nest JS
tags:
  - Nest JS
---

# Package.json 스크립트

## Jest

: 자바스크립트를 아주 쉽게 테스팅하는 npm 패키지

=> jest가 .spec.ts 파일을 찾아 볼 수 있도록 설정

* .spec.ts이라는 파일은 테스트를 포함함

  

1.  **cov(coverage) 스크립트 실행**

​	-> 코드가 얼마나 테스팅 되었는지

*  npm run test:cov

  -> 모든 spec파일을 찾아서 몇 줄이 테스팅되었는지 알려줌

  => **따라서 테스트하고 싶은 파일에 .spec.ts 붙이기!**



2. **watch mode 에서 테스팅**

   * npm run test:watch

     -> 모든 테스트파일을 찾아 실행함

## 테스팅 종류

1. **유닛 테스팅**

   -> **모든 function을 따로 테스트**(=서비스에서 분리된 유닛을 테스트)

   ex) getAll () 테스트

2. **end-to-end(e2e) 테스트**

   -> **모든 시스템을 테스팅**

   ex) 특정 페이지가 나오는지 확인

   

## 유닛 테스팅

1. 테스트하는 법

   * movies.service.spec.ts

     ```typescript
     import { Test, TestingModule } from '@nestjs/testing';
     import { MoviesService } from './movies.service';
     
     describe('MoviesService', () => {
       let service: MoviesService;
     
       beforeEach(async () => {
         const module: TestingModule = await Test.createTestingModule({
           providers: [MoviesService],
         }).compile();
     
         service = module.get<MoviesService>(MoviesService);
       });
     
       it('should be defined', () => {
         expect(service).toBeDefined();
       });
     
     
       it("should be 4", () => {
         expect(2+2).toEqual(4)// 2+2가 4와 같은지 기대(expect)
       })// 안의 텍스트는 중요하지 않음 & test하고 싶은 funtion
     });
     
     ```

     -> 콘솔창으로 확인

2. movies.service 테스팅

   1. getAll() 테스트

      * movies.service.spec.ts

        ```typescript
          describe("getAll",() => {
        
            it("should return an array", () => {
        
              const result = service.getAll(); //getAll함수 호출
        
              expect(result).toBeInstanceOf(Array); //배열로 반환하는지 테스트
            })
          })//꼭 함수와 같은 이름일 필요는 없음
        ```

   2. getOne() 테스트

      * movies.service.spec.ts

        ```typescript
          describe("getOne", () => {
            it("should return a movie", () => {
              service.create({  // movie 만들기
                title:"Test Movie",
                genres: ['test'],
                year: 2000,
              });
              const movie = service.getOne(1);
        
              expect(movie).toBeDefined();
              expect(movie.id).toEqual(1);
            });
            it("should throw 404 error", () => {
              try{
                service.getOne(999); //에러가 생기는 지 확인
              }catch(e){
                expect(e).toBeInstanceOf(NotFoundException) //에러가 NotFoundException인지 
                expect(e.message).toEqual("Movie with ID 999 not found."); //에러메세지가 잘 나오는지
              }
            })
          })
        ```

   3. deleteOne() 테스트

      * movies.service.spec.ts

        ```typescript
          describe("deleteOne", () => {
            it("deletes a movie", () => {
              service.create({
                title:"Test Movie",
                genres: ['test'],
                year: 2000,
              })
              const allMovies = service.getAll().length;
              service.deleteOne(1)
              const afterDelete = service.getAll().length;
        
              expect(afterDelete).toBeLessThan(allMovies); //제대로 삭제된게 맞는지 
            });
            it("should return a 404", () => {
              try{ 
                service.deleteOne(999);
              } catch (e) {
                expect(e).toBeInstanceOf(NotFoundException); //에러를 제대로 반환하는지
              }
            });
          });
        ```

   4. create 테스트

      * movies.service.spec.ts

        ```typescript
        describe("create", () => {
            it("should create a movie", () => {
              const beforeCreate = service.getAll().length
              service.create({
                title:"Test Movie",
                genres: ['test'],
                year: 2000,
              });
              const afterCreate = service.getAll().length;
              expect(afterCreate).toBeGreaterThan(beforeCreate); //생성된 것이 맞는지 
            });
          });
        ```

   5. update 테스트

      * movies.service.spec.ts

        ```typescript
          describe("update", () => {
            it("should update a movie", () => {
              service.create({
                title:"Test Movie",
                genres: ['test'],
                year: 2000,
              });
              service.update(1, {title:"Updated Test"});
              const movie= service. getOne(1);
              expect(movie.title).toEqual('Updated Test'); //title 검사
            });
            it("should return a 404", () => {
              try{ 
                service.update(999, {});
              } catch (e) {
                expect(e).toBeInstanceOf(NotFoundException); //에러를 제대로 반환하는지
              }
            });
          })
        ```

        

## E2E 테스팅

npm run test:e2e

1. get 테스트

* app.e2e-spec.ts

  ```typescript
    it("/movies (GET)", () => {
      return request(app.getHttpServer())
        .get("/movies") //movies에 GET 요청 테스트
        .expect(200)
        .expect([]);
    })
  ```

2. create & delete 테스트

   ```typescript
     describe("/movies (GET)", () => {
       it("/movies (GET)", () => {
         return request(app.getHttpServer())
           .get("/movies") //movies에 GET 요청 테스트
           .expect(200)
           .expect([]);
       });
   
       it("POST", () => {
         return request(app.getHttpServer())
           .post('/movies')
           .send({
             title:"TEST",
             year: 2000,
             genres: ['test'],
           })
           .expect(201); //서버에 request해서 movies에 post할때 위 정보들을 보내면 201을 받는지
       });
   
       it("DELETE", () => {
         return request(app.getHttpServer())
           .delete('/movies')
           .expect(404);
       });
     });
   ```

3. id의 endpoint

   * 테스팅 시작전 새 어플리케이션 만들기

     -> 테스팅 할때마다 어플리케이션을 매번 실행하지 않도록

     ```typescript
       describe('/movies/:id',() => {
         it.todo('GET 200', () => {
           return request(app.getHttpServer())
             .get("/movies/1")
             .expect(200)
         });
         it.todo('DELETE');
         it.todo('PATCH');
       })
     ```

     =>에러가 남

     -> 실제 사용 서버에선 id 타입이 number지만 테스팅 서버에선 string으로 나타나기 때문

     -> main에서 controller 타입을 자동으로 바꿨기 때문에!

     **=> pipe 쓰기 **

     ```typescript
       beforeAll(async () => { //beforeAll로 매번 새 어플리케이션 생성하지 않도록
         const moduleFixture: TestingModule = await Test.createTestingModule({
           imports: [AppModule],
         }).compile();
     
         app = moduleFixture.createNestApplication();
         app.useGlobalPipes(
           new ValidationPipe({
             whitelist:true,
             forbidNonWhitelisted:true,
             transform: true
             }), //ValidationPipe가 유효성 검사
           );
         await app.init();
       });
     
     ```

     * GET, PATCH, DELETE 테스트

       ```typescript
         describe('/movies/:id',() => { //
           it('GET 200', () => {
             return request(app.getHttpServer())
               .get("/movies/1")
               .expect(200)
           });
           
           it('PATCH', () => {
             return request(app.getHttpServer())
               .patch('/movies/1')
               .send({ title: 'Updated Test'})
               .expect(200);
           });
           it('DELETE', () => {
             return request(app.getHttpServer())
               .delete('/movies/1')
               .expect(200);
           });
         })
       });
       
       ```

     * 잘못된 데이터 create 테스트

       ```typescript
         describe("/movies (GET)", () => {
           it("/movies (GET)", () => {
             return request(app.getHttpServer())
               .get("/movies") //movies에 GET 요청 테스트
               .expect(200)
               .expect([]);
           });
       
           it("POST", () => {
             return request(app.getHttpServer())
               .post('/movies')
               .send({
                 title:"TEST",
                 year: 2000,
                 genres: ['test'],
               })
               .expect(201); //서버에 request해서 movies에 post할때 위 정보들을 보내면 201을 받는지
           });
       
           it("POST 400", () => {
             return request(app.getHttpServer())
               .post('/movies')
               .send({
                 title:"TEST",
                 year: 2000,
                 genres: ['test'],
                 other:"thing"
               })
               .expect(400); //서버에 request해서 movies에 post할때 위 정보들을 보내면 201을 받는지
           }); 
       
           it("DELETE", () => {
             return request(app.getHttpServer())
               .delete('/movies')
               .expect(404);
           });
         });
       ```

       
