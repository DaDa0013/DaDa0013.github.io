---
title: "우버이츠클론코딩 #9 Unit Testing JWT and Mail"
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
excerpt: "testing jwt"
---

# JWT Serivce test

### setup

* 외부 라이브러리 mocking 

* jwt.service.spec.ts

  ```typescript
  import { Test } from '@nestjs/testing';
  import { CONFIG_OPTIONS } from 'src/common/common.constant';
  import { JwtService } from './jwt.service';
  
  describe('JwtService', () => {
    let service: JwtService;
    beforeEach(async () => {
      const module = await Test.createTestingModule({
        providers: [
          JwtService,
          {
            //dependency
            provide: CONFIG_OPTIONS,
            useValue: { privateKey: 'testKey' },
          },
        ],
      }).compile();
      service = module.get<JwtService>(JwtService);
    });
  
    it('should be defined', () => {
      expect(service).toBeDefined();
    });
  });
  
  ```

### sign test

* sign token 반환 test(token)

```typescript
//2개 반환하는 함수 mocking
//dependency mocking 
jest.mock('jsonwebtoken', () => {
  return {
    sign: jest.fn(() => 'TOKEN'),
  };
});
...

  describe('sign', () => {
    it('should return a signed token', () => {
      const ID = 1;
      const token = service.sign(ID);
      expect(typeof token).toBe('string');//service.sign 반환값 체크
      expect(jwt.sign).toHaveBeenCalledTimes(1);
      expect(jwt.sign).toHaveBeenLastCalledWith({ id: ID }, TEST_KEY);
    });
  });
  describe('verify', () => {
    it('should return the decoded token', () => {});
  });
```

### verify test

```typescript
  describe('verify', () => {
    it('should return the decoded token', () => {
        const TOKEN = 'TOKEN';
        const decodedToken = service.verify(TOKEN);
        expect(decodedToken).toEqual({ id: USER_ID });
        expect(jwt.verify).toHaveBeenCalledTimes(1);
        expect(jwt.verify).toHaveBeenCalledWith(TOKEN, TEST_KEY);
      });
  });
});
```



# Mail test

```typescript
import { Test } from '@nestjs/testing';
import { CONFIG_OPTIONS } from 'src/common/common.constant';
import { MailService } from './mail.service';

jest.mock('got', () => {});
jest.mock('form-data', () => {
  return {
    append: jest.fn(),
  };
});

describe('MailService', () => {
  let service: MailService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        MailService,
        {
          provide: CONFIG_OPTIONS,
          useValue: {
            apiKey: 'test-apiKey',
            domain: 'test-domain',
            fromEmail: 'test-fromEmail',
          },
        },
      ],
    }).compile();
    service = module.get<MailService>(MailService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  it.todo('sendEmail');
  it.todo('sendVerificationEmail');
});

```

### sendVerificationEmail Test

-> sendEmail을 mock 하지 않고 spying하기

```typescript
import { Test } from '@nestjs/testing';
import { CONFIG_OPTIONS } from 'src/common/common.constant';
import { MailService } from './mail.service';

jest.mock('got', () => {});
jest.mock('form-data', () => {
  return {
    append: jest.fn(),
  };
});

describe('MailService', () => {
  let service: MailService;
  //jest로 이미 받았기 때문에 import 안함
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        MailService,
        {
          provide: CONFIG_OPTIONS,
          useValue: {
            apiKey: 'test-apiKey',
            domain: 'test-domain',
            fromEmail: 'test-fromEmail',
          },
        },
      ],
    }).compile();
    service = module.get<MailService>(MailService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  describe('sendVerificationEmail', () => {
    it('should call sendEmail', () => {
      const sendVerificationEmailArgs = {
        email: 'email',
        code: 'code',
      };
      jest.spyOn(service, 'sendEmail').mockImplementation(async () => {}); //sendemail spying
      service.sendVerificationEmail(
        sendVerificationEmailArgs.email,
        sendVerificationEmailArgs.code,
      );
      expect(service.sendEmail).toHaveBeenCalledTimes(1);
      expect(service.sendEmail).toHaveBeenCalledWith(
        'verify your Email',
        'hi',
        [
          { key: 'code', value: sendVerificationEmailArgs.code },
          { key: 'username', value: sendVerificationEmailArgs.email },
        ],
      );
    });
  });

  it.todo('sendEmail');
});

```



### sendEmail

* formdata 호출 확인
* got이 string과 object 가지고 실행되는지 확인

```typescript

  describe('sendEmail', () => {
    it('sends email', async () => {
      const ok = await service.sendEmail('', '', []); //subject, template, 변수 안에 넣기
      const formSpy = jest.spyOn(FormData.prototype, 'append');
      expect(formSpy).toHaveBeenCalled();
      expect(got.post).toHaveBeenCalledTimes(1);
      expect(got.post).toHaveBeenCalledWith(
        `https://api.mailgun.net/v3/${TEST_DOMAIN}/messages`,
        expect.any(Object),
      );
        //TEST_DOMAIIN이 url로 호출한 got까지 내려갔는지 확인
      expect(ok).toEqual(true);
    });
    it('fails on error', async () => {
      jest.spyOn(got, 'post').mockImplementation(() => {
        throw new Error();
      });
      const ok = await service.sendEmail('', '', []);
      expect(ok).toEqual(false);
    });
  });
```

