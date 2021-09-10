---
title: "우버이츠클론코딩 #7 email verification"
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
---

## User verification

* verification.entity

```typescript
import { Field, InputType, ObjectType } from '@nestjs/graphql';
import { CoreEntity } from 'src/common/entities/core.entity';
import { Column, Entity, JoinColumn, OneToOne } from 'typeorm';
import { User } from './user.entity';

@InputType({ isAbstract: true })
@ObjectType()
@Entity()
export class Verification extends CoreEntity {
  //id, createdAt, updatedAt 가지고 있음(CoreEntity)
  @Column()
  @Field((type) => String)
  code: string;

  //One-to-one relations -> user와 verification는 일대일 대응
  //user를 통해 verification에 접근하거나 그 verification으로부터 user에 접근하려면 JoinColumn()을 추가해야함(어느쪽에 추가해야할지 주의)
  @OneToOne((type) => User)
  @JoinColumn()
  user: User;
}

```

* user.entity.ts

  ```typescript
      @Column({default: false})
      @Field(type => Boolean)
      verifield: boolean; // User의 emial이 verifiy됐는지 안됐는지 저장하는 용도
  ```

  

* users.service.ts

  -> 계정 생성시 verification 하도록

  ```typescript
  @Injectable()
  export class UsersService {
    constructor(
      @InjectRepository(User) private readonly users: Repository<User>, //User entity의 InjectRepository 불러오기 & type이 repository이고 repository type은 user enitity
      @InjectRepository(Verification)
      private readonly verifications: Repository<Verification>,
      private readonly jwtService: JwtService, //nestjs는 클래스 타입만 보고 import 알아서 찾아줌
      private readonly config: ConfigService,
    ) {
      this.jwtService.hello();
    }
  
    async createAccount({
      email,
      password,
      studentId,
      role,
    }: CreateAccountInput): Promise<{ ok: boolean; error?: string }> {
      //check new user(that email does not exist)
      try {
        const exists = await this.users.findOne({ email }); //findOne = 주어진 condition(환경)과 일치하는 첫 번째 entity 찾기
        const exists2 = await this.users.findOne({ studentId });
        if (exists || exists2) {
          //make error
          return { ok: false, error: '이미 가입한 이메일이거나 학번입니다.' }; //boolean =false, error ="there~"
        }
        const user = await this.users.save(
          this.users.create({ email, password, studentId, role }),
        ); //없다면 새로운 계정 create & save
        await this.verifications.save(
          this.verifications.create({
            //verification과 user를 저장
            user,
          }),
        );
        return { ok: true };
      } catch (e) {
        console.log(e);
        return { ok: false, error: "Couldn't create account" };
      }
      // create user & hash the password
    }
    
  ```

* users.service.ts

  ```typescript
    async editProfile(userId: number, { email, password }: EditProfileInput) {
      const user = await this.users.findOne(userId);
      if (email) {
        user.email = email;
        user.verifield = false; // user를 verified 하기
        await this.verification.save(this.verification.create({ user }));
      }
      if (password) {
        user.password = password;
      }
      return this.users.save(user); //db에 entity 존재 유무 체크 안함
    } // save는 entity 없으면 생성함
  
  ```



## verify user

-> verification code 를 사용해서 그들의 verification 찾기

=> 지우고 user에 대한 verify 하기

* users.resolver.ts

  ```typescript
  
    @Mutation((returns) => VerifyEmailOutput)
    verifyEmail(@Args('input') {code}: VerifyEmailInput) {
      this.usersService.verifyEmail(code);
    }
  ```
  
* users.service.ts

  ```typescript
    //verification가 존재하면 삭제, verification과 연결된 user를 찾아 verified를 true로 변경
    async verifyEmail(code: string): Promise<boolean> {
      //verification찾기
      const verification = await this.verification.findOne({ code });
      if (verification) {
        console.log(verification, verification.user);
      }
      return false;
    }
  ```

  => typeORM은 relationship를 찾아주지않음

  => TypeORM에게 요구하기

* users.service.ts

  ```typescript
    async verifyEmail(code: string): Promise<boolean> {
      //verification찾기
      const verification = await this.verification.findOne(
        { code },
        { relations: ['user'] },
      ); //relations를 통해 실제로 불러오고자 하는 관계를 적을 수 있음 -> 여기선 user class를
      if (verification) {
        verification.user.verifield = true
        this.users.save(verification.user)
      }
      return false;
    }
  }
  ```

  

위에서 this.users.save떄문에 hash된 password를 한번 더 hash 함

1. User에 password를 포함하지 않기

   * users.entity.ts

     ```typescript
       @Column({select: false})
       @Field((type) => String)
       password: string;
     ```

2. password가 있을 경우에만 password를 hash 하도록

   * users.entity.ts

     ```typescript
       @BeforeInsert() //DB에 저장하기 전에 password hash해주기
       async hashPassword(): Promise<void> {
         if(this.password){
           try {
             this.password = await bcrypt.hash(this.password, 10); //hash round는 10으로
           } catch (e) {
             console.log(e);
             throw new InternalServerErrorException();
           }
         }
       }
     ```



* verification.entity.ts

  => user를 삭제하면 verification도 같이 삭제하도록

  ```typescript
    @OneToOne((type) => User, {onDelete:"CASCADE"})
    @JoinColumn()
    user: User;
  ```



## Mailgun Setup

-> 이메일 모듈 만들어서 유저 인증

* Mailgun: 이메일 보내는 서비스

  * 계정 생성하기
  * mail module 생성

* env.dev

  -> 전에 port나 password한 거처럼

  ```typescript
      MAILGUN_API_KEY=e55a7958b199abaea470bd7209c227ca-a3c55839-23263429
      MAILGUN_DOMAIN_NAME=sandbox330a06e3217344a99800f0b0d1674f2b.mailgun.org
      MAILGUN_FROM_EMAIL=nuber@eats@nomadcoders.co
  ```

  

## MailGun API

* 메일 서비스 만들기

1. config injection 작동 체크

   * mail.service.ts

     ```typescript
     import { Global, Inject, Injectable } from '@nestjs/common';
     import { CONFIG_OPTIONS } from 'src/common/common.constant';
     import { MailModuleOptions } from './mail.interfaces';
     
     @Injectable()
     export class MailService {
       constructor(
         @Inject(CONFIG_OPTIONS) private readonly options: MailModuleOptions,
       ) {
         console.log(options);
       }
     }
     
     ```

     => mail.module에서 MailService 추가

## email 템플릿

* mailgun sending의 Templates

  -> 템플릿 하나 고르기

  ```css
  <!DOCTYPE html>
  <html  style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
  <head>
  <meta name="viewport" content="width=device-width" />
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  <title>Confirm Nuber Eats Account</title>
  <style type="text/css">
  img {
  max-width: 100%;
  }
  body {
  -webkit-font-smoothing: antialiased; -webkit-text-size-adjust: none; width: 100% !important; height: 100%; line-height: 1.6em;
  }
  body {
  background-color: #f6f6f6;
  }
  @media only screen and (max-width: 640px) {
    body {
      padding: 0 !important;
    }
    h1 {
      font-weight: 800 !important; margin: 20px 0 5px !important;
    }
    h2 {
      font-weight: 800 !important; margin: 20px 0 5px !important;
    }
    h3 {
      font-weight: 800 !important; margin: 20px 0 5px !important;
    }
    h4 {
      font-weight: 800 !important; margin: 20px 0 5px !important;
    }
    h1 {
      font-size: 22px !important;
    }
    h2 {
      font-size: 18px !important;
    }
    h3 {
      font-size: 16px !important;
    }
    .container {
      padding: 0 !important; width: 100% !important;
    }
    .content {
      padding: 0 !important;
    }
    .content-wrap {
      padding: 10px !important;
    }
    .invoice {
      width: 100% !important;
    }
  }
  </style>
  </head>
  
  <body itemscope itemtype="http://schema.org/EmailMessage" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; -webkit-font-smoothing: antialiased; -webkit-text-size-adjust: none; width: 100% !important; height: 100%; line-height: 1.6em; background-color: #f6f6f6; margin: 0;" bgcolor="#f6f6f6">
  
  <table class="body-wrap" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; width: 100%; background-color: #f6f6f6; margin: 0;" bgcolor="#f6f6f6"><tr style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><td style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0;" valign="top"></td>
      <td class="container" width="600" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; display: block !important; max-width: 600px !important; clear: both !important; margin: 0 auto;" valign="top">
        <div class="content" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; max-width: 600px; display: block; margin: 0 auto; padding: 20px;">
          <table class="main" width="100%" cellpadding="0" cellspacing="0" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; border-radius: 3px; background-color: #fff; margin: 0; border: 1px solid #e9e9e9;" bgcolor="#fff"><tr style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><td class="alert alert-warning" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 16px; vertical-align: top; color: #fff; font-weight: 500; text-align: center; border-radius: 3px 3px 0 0; background-color: #FF9F00; margin: 0; padding: 20px;" align="center" bgcolor="#FF9F00" valign="top">
                Please Confirm Your Account
              </td>
            </tr><tr style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><td class="content-wrap" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 20px;" valign="top">
                <table width="100%" cellpadding="0" cellspacing="0" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><tr style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><td class="content-block" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 0 0 20px;" valign="top">
                      Hello {{username}}!
                    </td>
                  </tr><tr style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><td class="content-block" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 0 0 20px;" valign="top">
                      Please confirm your account!
                    </td>
                  </tr><tr style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><td class="content-block" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 0 0 20px;" valign="top">
                      <a href="http://127.0.0.1:3000/confirm?code={{code}}" class="btn-primary" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; color: #FFF; text-decoration: none; line-height: 2em; font-weight: bold; text-align: center; cursor: pointer; display: inline-block; border-radius: 5px; text-transform: capitalize; background-color: #348eda; margin: 0; border-color: #348eda; border-style: solid; border-width: 10px 20px;">Upgrade my account</a>
                    </td>
                  </tr><tr style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><td class="content-block" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 0 0 20px;" valign="top">
                      Thanks for choosing Nuber Eats
                    </td>
                  </tr></table></td>
            </tr></table><div class="footer" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; width: 100%; clear: both; color: #999; margin: 0; padding: 20px;">
            <table width="100%" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><tr style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;"><td class="aligncenter content-block" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 12px; vertical-align: top; color: #999; text-align: center; margin: 0; padding: 0 0 20px;" align="center" valign="top"><a href="http://www.mailgun.com" style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 12px; color: #999; text-decoration: underline; margin: 0;">Unsubscribe</a> from these alerts.</td>
              </tr></table></div></div>
      </td>
      <td style="font-family: 'Helvetica Neue',Helvetica,Arial,sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0;" valign="top"></td>
    </tr></table></body>
  </html>
  ```

  ![image](https://user-images.githubusercontent.com/79195793/132830295-8bd20da2-4d5d-4fc9-95d4-113805d7191a.png)

![image](https://user-images.githubusercontent.com/79195793/132830320-1c61b44c-2e2a-47f5-81fc-ca283e68e092.png)

* mail.service.ts

  ```typescript
  import { Global, Inject, Injectable } from '@nestjs/common';
  import { CONFIG_OPTIONS } from 'src/common/common.constant';
  import { EamilVar, MailModuleOptions } from './mail.interfaces';
  import got from 'got';
  import * as FormData from 'form-data';
  
  @Injectable()
  export class MailService {
    constructor(
      @Inject(CONFIG_OPTIONS) private readonly options: MailModuleOptions,
    ) {}
    //email 전송하는 함수
    private async sendEmail(
      subject: string,
      template: string,
      emailVars: EamilVar[],
    ) {
      const form = new FormData();
      form.append(
        'from',
        `Nico from Nuber Eats <mailgun@${this.options.domain}>`,
      ); // 보내는 사람
      form.append('to', `yoon351200@naver.com`); // 받는 사람
      form.append('subject', subject);
      form.append('template', template);
      emailVars.forEach((eVar) => form.append(`v:${eVar.key}`, eVar.value));
      try {
        const response = await got(
          `https://api.mailgun.net/v3/${this.options.domain}/messages`,
          {
            method: 'POST',
            headers: {
              Authorization: `Basic ${Buffer.from(
                `api:${this.options.apiKey}`,
              ).toString('base64')}`,
            },
            body: form,
          },
        );
      } catch (error) {
        console.log(error);
      }
    }
    //sendEmail 함수 호출
    sendVerificationEmail(email: string, code: string) {
      this.sendEmail('verify your Email', 'hi', [
        { key: 'code', value: code },
        { key: 'username', value: email },
      ]);
    }
  }
  
  ```

  ![image](https://user-images.githubusercontent.com/79195793/132830333-ebf41ba3-144a-4b0a-82c4-a5e1183531df.png)

  -> 내 메일로 옴

  

  > pgadmin의 table 칼럼 수정해야함!

  
