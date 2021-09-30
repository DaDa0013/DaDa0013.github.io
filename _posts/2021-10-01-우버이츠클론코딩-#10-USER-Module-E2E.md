---
title: "우버이츠클론코딩 #10 USER Module E2E"
categories:
  - clone coding
  - PostgreSQL
tags:
  - clone coding
  - PostgreSQL
  - Nest JS
  - typeORM
toc: true  
toc_sticky: true 
excerpt: "user Module"
---

# USER Module Test

* users.e2e-spec.ts

  ```typescript
  import { Test, TestingModule } from '@nestjs/testing';
  import { INestApplication } from '@nestjs/common';
  import * as request from 'supertest';
  import { AppModule } from '../src/app.module';
  
  describe('UserModule (e2e)', () => {
    let app: INestApplication;
  
    beforeEach(async () => {
      const module: TestingModule = await Test.createTestingModule({
        imports: [AppModule],
      }).compile();
  
      app = module.createNestApplication();
      await app.init();
    });
  });
  
  ```

  -> 경로 에러 & environment 파일 load 안됨

* jest-e2e.json

  -> 경로 에러 수정

  ```typescript
  {
    "moduleFileExtensions": ["js", "json", "ts"],
    "rootDir": ".",
    "testEnvironment": "node",
    "testRegex": ".e2e-spec.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "moduleNameMapper": {
      "^src/(.*)$": "<rootDir>/../src$1"
    }
  }
  
  ```

* .env.test

  -> configModule에 envFilePath의 2가지 경로(dev, test)

  => test파일 수정

  ```typescript
      
      DB_HOST=localhost
      DB_PORT=5432
      DB_USERNAME=postgres
      DB_PASSWORD=1234
      DB_NAME=nuber-eats-test
      PRIVATE_KEY=testKey
      MAILGUN_API_KEY=e55a7958b199abaea470bd7209c227ca-a3c55839-23263429
      MAILGUN_DOMAIN_NAME=sandbox330a06e3217344a99800f0b0d1674f2b.mailgun.org
      MAILGUN_FROM_EMAIL=nuber@eats@nomadcoders.co
  ```

  

* users.e2e-spec.ts

  ```
  import { Test, TestingModule } from '@nestjs/testing';
  import { INestApplication } from '@nestjs/common';
  import * as request from 'supertest';
  import { AppModule } from '../src/app.module';
  import { SSL_OP_ALLOW_UNSAFE_LEGACY_RENEGOTIATION } from 'constants';
  
  describe('UserModule (e2e)', () => {
    let app;
  
    beforeAll(async () => {
      //beforeAll: 모든 test전에 module load
      const module: TestingModule = await Test.createTestingModule({
        imports: [AppModule],
      }).compile();
  
      app = module.createNestApplication();
      await app.init();
    });
  
    it.todo('me');
    it.todo('userProfile');
    it.todo('createAccount');
    it.todo('login');
    it.todo('editProfile');
    it.todo('verifyEmail');
  });
  
  ```

  ->이떄 .env.test의 데이터베이스 생성해야함

* users.e2e-spec.ts

  -> 어플리케이션 종료

  ```typescript
  import { INestApplication } from '@nestjs/common';
  import { Test, TestingModule } from '@nestjs/testing';
  import { getConnection } from 'typeorm';
  import { AppModule } from '../src/app.module';
  
  const GRAPHQL_ENDPOINT = '/graphql';
  
  describe('UserModule (e2e)', () => {
    let app: INestApplication;
  
    beforeAll(async () => {
      //beforeAll: 모든 test전에 module load
      const module: TestingModule = await Test.createTestingModule({
        imports: [AppModule],
      }).compile();
  
      app = module.createNestApplication();
      await app.init();
    });
  
    //application 종료
    afterAll(async () => {
      await getConnection().dropDatabase(); //데이터베이스랑 connect
      app.close();
    });
  
    it.todo('createAccount');
    it.todo('userProfile');
    it.todo('login');
    it.todo('me');
    it.todo('verifyEmail');
    it.todo('editProfile');
  });
  
  ```

  

### test createAccount

-> createaccount test

```typescript
describe('createAccount', () => {
     const EMAIL = 'nico@las.com';
     it('should create account', () => {
       return request(app.getHttpServer())
         .post(GRAPHQL_ENDPOINT)
         .send({
           query: `
           mutation {
             createAccount(input: {
               email:"${EMAIL}",
               password:"12345",
               role:Owner
             }) {
               ok
               error
             }
           }
           `,
         })
         .expect(200)
         .expect(res => {
           expect(res.body.data.createAccount.ok).toBe(true);
           expect(res.body.data.createAccount.error).toBe(null);
         });
     });

     it.todo('should fail if account already exists');
   });
```

-> 계정이 이미 있으면 실패하도록

```typescript
    it('should fail if account already exists', () => {
      return request(app.getHttpServer())
        .post(GRAPHQL_ENDPOINT)
        .send({
          query: `
          mutation {
            createAccount(input: {
              email:"${EMAIL}",
              password:"12345",
              role:Owner
            }) {
              ok
              error
            }
          }
        `,
        })
        .expect(200)
        .expect(res => {
          expect(res.body.data.createAccount.ok).toBe(false);
          expect(res.body.data.createAccount.error).toBe(
            'There is a user with that email already',
          );
        });
    });
```



### Testing login

```typescript
    it('should not be able to login with wrong credentials', () => {
      return request(app.getHttpServer())
        .post(GRAPHQL_ENDPOINT)
        .send({ //잘못된 password
          query: `
          mutation {
            login(input:{
              email:"${testUser.email}",
              password:"xxx",
            }) {
              ok
              error
              token
            }
          }
        `,
        })
        .expect(200)
        .expect((res) => {
          const {
            body: {
              data: { login },
            },
          } = res;
          expect(login.ok).toBe(false);
          expect(login.error).toBe('잘못된 비밀번호입니다');
          expect(login.token).toBe(null);
        });
    });
  });
```

### userProfile

-> database를 계속 재시작하기 떄문에 user는 항상 1

```typescript
  describe('userProfile', () => {
    let userId: number;
    beforeAll(async () => {
      const [user] = await usersRepository.find(); //user 첫번째
      userId = user.id;
    });
    it("should see a user's profile", () => {
      return request(app.getHttpServer())
        .post(GRAPHQL_ENDPOINT)// post 뒤에 header를 set해야함
        .set('X-JWT', jwtToken) //header, value
        .send({
          query: `
        {
          userProfile(userId:${userId}){
            ok
            error
            user {
              id
            }
          }
        }
        `,
        })
        .expect(200)
        .expect((res) => {
          const {
            body: {
              data: {
                userProfile: {
                  ok,
                  error,
                  user: { id },
                },
              },
            },
          } = res;
          expect(ok).toBe(true);
          expect(error).toBe(null);
          expect(id).toBe(userId);
        });
    });
    it('should not find a profile', () => {
      return request(app.getHttpServer())
        .post(GRAPHQL_ENDPOINT)
        .set('X-JWT', jwtToken)
        .send({
          query: `
        {
          userProfile(userId:666){
            ok
            error
            user {
              id
            }
          }
        }
        `,
        })
        .expect(200)
        .expect((res) => {
          const {
            body: {
              data: {
                userProfile: { ok, error, user },
              },
            },
          } = res;
          expect(ok).toBe(false);
          expect(error).toBe('User Not Found');
          expect(user).toBe(null);
        });
    });
```



### Testing me

* 로그인 됐을 떄 옵션
* 로그인 안됐을때

```typescript
describe('me', () => {
     it('should find my profile', () => {
       return request(app.getHttpServer())
         .post(GRAPHQL_ENDPOINT)
         .set('X-JWT', jwtToken)
         .send({
           query: `
           {
             me {
               email
             }
           }
         `,
         })
         .expect(200)
         .expect(res => {
           const {
             body: {
               data: {
                 me: { email },
               },
             },
           } = res;
           expect(email).toBe(testUser.email);
         });
     });
     it('should not allow logged out user', () => {
       return request(app.getHttpServer())
         .post(GRAPHQL_ENDPOINT)
         .send({
           query: `
         {
           me {
             email
           }
         }
       `,
         })
         .expect(200)
         .expect(res => {
           const {
             body: { errors },
           } = res;
           const [error] = errors;
           expect(error.message).toBe('Forbidden resource');
         });
     });
   });
```



### editProfile tesing

* email, password 바꾸기
* 새로운 email 만들기

```typescript
  describe('editProfile', () => {
    //새 이메일이 데이터베이스에 있는지
    const NEW_EMAIL = 'nico@new.com';
    it('should change email', () => {
      return request(app.getHttpServer())
        .post(GRAPHQL_ENDPOINT)
        .set('X-JWT', jwtToken)
        .send({
          query: `
            mutation {
              editProfile(input:{
                email: "${NEW_EMAIL}"
              }) {
                ok
                error
              }
            }
        `,
        })
        .expect(200)
        .expect((res) => {
          const {
            body: {
              data: {
                editProfile: { ok, error },
              },
            },
          } = res;
          expect(ok).toBe(true);
          expect(error).toBe(null);
        });
    });

    it('should have new email', () => {
      return request(app.getHttpServer())
        .post(GRAPHQL_ENDPOINT)
        .set('X-JWT', jwtToken)
        .send({
          query: `
          {
            me {
              email
            }
          }
        `,
        })
        .expect(200)
        .expect((res) => {
          const {
            body: {
              data: {
                me: { email },
              },
            },
          } = res;
          expect(email).toBe(NEW_EMAIL);
        });
    });
  });
```

-> 이때 user가 먼저 생성돼서 verification이 있는데 또 생성될 수 있으므로 user.service.ts에서 verification을 삭제하는 코드 추가하기



### verifyEmail testing

-> 새 database를 만들면 verification이 하나 생기고 그 verification을 삭제해야함!

```typescript
  describe('editProfile', () => {
    //새 이메일이 데이터베이스에 있는지
    const NEW_EMAIL = 'nico@new.com';
    it('should change email', () => {
      return request(app.getHttpServer())
        .post(GRAPHQL_ENDPOINT)
        .set('X-JWT', jwtToken)
        .send({
          query: `
            mutation {
              editProfile(input:{
                email: "${NEW_EMAIL}"
              }) {
                ok
                error
              }
            }
        `,
        })
        .expect(200)
        .expect((res) => {
          const {
            body: {
              data: {
                editProfile: { ok, error },
              },
            },
          } = res;
          expect(ok).toBe(true);
          expect(error).toBe(null);
        });
    });

    it('should have new email', () => {
      return request(app.getHttpServer())
        .post(GRAPHQL_ENDPOINT)
        .set('X-JWT', jwtToken)
        .send({
          query: `
          {
            me {
              email
            }
          }
        `,
        })
        .expect(200)
        .expect((res) => {
          const {
            body: {
              data: {
                me: { email },
              },
            },
          } = res;
          expect(email).toBe(NEW_EMAIL);
        });
    });
  });
```

