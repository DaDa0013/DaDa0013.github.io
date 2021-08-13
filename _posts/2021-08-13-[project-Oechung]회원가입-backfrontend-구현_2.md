---
title: "[project Oechung]회원가입 backfrontend 구현_2"
categories:
  - Backend
  - Project
  - PostgreSQL
tags:
  - NestJs
  - postgreSQL
  - typescript
toc: true
toc_sticky: true
excerpt: "backend -(delete)"
---

# 구현한 기능

* 회원 계정 삭제

* 회원 계정 삭제 일자

  * users.resolver.ts

  ```typescript
  
  import { UseGuards } from "@nestjs/common";
  import { Resolver, Query, Mutation, Args, Context} from "@nestjs/graphql";
  import { AuthUser } from "src/auth/auth-user.decorator";
  import { AuthGuard } from "src/auth/auth.guard";
  import { UserProfileInput, UserProfileOutput } from "src/users/dtos/user-profile.dto";
  import { CreateAccountInput, CreateAccountOutput } from "./dtos/create-account.dto";
  import { DeleteAccountInput, DeleteAccountOutput } from "./dtos/delete-account.dto";
  import { EditProfileInput, EditProfileOutput } from "./dtos/edit-profile.dto";
  import { LoginInput, LoginOutput} from "./dtos/login.dto";
  import { User } from "./entities/user.entity";
  import { UsersService } from "./users.service";
  
  @Resolver(of => User)
  export class UsersResolver {
      constructor(
          private readonly usersService: UsersService
      ){}
  
      @Query(returns => Boolean)//graphQL 루트 만들기
      hi(){
          return true;
      }
  
      @Mutation(returns =>CreateAccountOutput)
      async createAccount(
          @Args("input") createAccountInput: CreateAccountInput,
      ): Promise <CreateAccountOutput>{ //createAccountInput이라는 input type만듦
          try{
              return this.usersService.createAccount(createAccountInput);
          } catch(error) {
              //에러 발생시
              return{
                  error,
                  ok: false,
              }
          }
      }
  
      @Mutation(returns => LoginOutput)
      async login(@Args('input') loginInput: LoginInput ): Promise<LoginOutput>{//input Arguments 필요
          try {
              return this.usersService.login(loginInput) //loginInput 저장
          } catch(error){
              return{
                  ok: false,
                  error,
              };
          }
      }
      @Query(returns => User)
      @UseGuards(AuthGuard)
      me(@AuthUser() authUser: User){
          return authUser;
      }//middleware 구현 //로그인한 사람이 누구인지 반환
  
      @UseGuards(AuthGuard)
      @Query(returns => UserProfileOutput)
      async userProfile(@Args()userProfileInput: UserProfileInput): Promise<UserProfileOutput>{ //user id가져오기
          try {
              const user = await this.usersService.findById(userProfileInput.userId)
              if (!user){
                  throw Error(); //user 찾지 못하면 error로
              }
              return{
                  ok:true,
                  user,
              };
          }catch(e){
              return{
                  error:"User Not Found",
                  ok: false,
              }
          };     
      }
      @UseGuards(AuthGuard)
      @Mutation(returns=> EditProfileOutput)
      async editProfile(@AuthUser() authUser: User, @Args('input') editProfileInput: EditProfileInput,
      ): Promise<EditProfileOutput>{
          try{
              await this.usersService.editProfile(authUser.id, editProfileInput);
              return {
                  ok:true,
              }
          }catch(error)
          {
              return  {
                  ok: false,
                  error
              }
          }
  
      }
      @UseGuards(AuthGuard)
      @Mutation(returns=> DeleteAccountOutput)
      async deleteAccount(
          @AuthUser() authUser: User,
          @Args('input') deleteAccountInput: DeleteAccountInput,
      ): Promise<DeleteAccountOutput>{
          try{
              await this.usersService.deleteAccount(authUser.id, deleteAccountInput);
              return {
                  ok: true,
              }
          }catch(error)
          {
              return  {
                  ok: false,
                  error
                  
              }
          }
  
      }
      
  }
  ```

  * delete-account.dto.ts

    ```typescript
    import { InputType, ObjectType, PartialType, PickType } from "@nestjs/graphql";
    import { CoreOutput } from "src/common/dtos/output.dto";
    import { User } from "../entities/user.entity";
    
    @ObjectType()
    export class DeleteAccountOutput extends CoreOutput {}
    
    @InputType()
    export class DeleteAccountInput extends PartialType(
        PickType(User, ["email","password", "studentId"]),//user에서 email과 password가지고 class 생성 & PartialType으로 optional하게 
    ){}
    
    ```

  * users.service.ts

    ```typescript
    import { Injectable, Query, RequestMethod } from "@nestjs/common";
    import { InjectRepository } from "@nestjs/typeorm";
    import { Repository } from "typeorm";
    import { CreateAccountInput } from "./dtos/create-account.dto";
    import { LoginInput } from "./dtos/login.dto";
    import { User } from "./entities/user.entity";
    import * as jwt from 'jsonwebtoken';
    import { ConfigService } from "@nestjs/config";
    import { JwtService } from "src/jwt/jwt.service";
    import { EditProfileInput } from "./dtos/edit-profile.dto";
    import { DeleteAccountInput } from "./dtos/delete-account.dto";
    import { userInfo } from "os";
    
    @Injectable()
    export class UsersService{
        constructor(
            @InjectRepository(User) private readonly users: Repository <User>,//User entity의 InjectRepository 불러오기 & type이 repository이고 repository type은 user enitity
            private readonly jwtService: JwtService, //nestjs는 클래스 타입만 보고 import 알아서 찾아줌
            ){}
    
        async createAccount({
            email,
            studentId,
            password, 
            role,
        }: CreateAccountInput): Promise <{ok: boolean, error? :string }>{
            //check new user(that email does not exist)
            try {
                const exists = await this.users.findOne({email}) //findOne = 주어진 condition(환경)과 일치하는 첫 번째 entity 찾기
                const exists2 = await this.users.findOne({studentId})
                if(exists || exists2){
                    //make error
                    return {ok: false, error: '이미 가입한 이메일이거나 학번입니다.'}; //boolean =false, error ="there~"
                }
                await this.users.save(this.users.create({email, studentId, password, role})); //없다면 새로운 계정 create & save
                return {ok: true};
            } catch(e){
                console.log(e);
                return {ok: false, error: "Couldn't create account"};
            }
            // create user & hash the password
            
        }
    
        async login({
            email, 
            password
        }: LoginInput): Promise<{ok: boolean; error?:string, token?: string}> {
            //make a JWT and give it to the user
            try{
                // find the user with the email
                const user = await this.users.findOne({ email });
                if(!user){ //user가 존재하지 않는다면
                    return {
                        ok:false,
                        error: 'User not found',
                    }
                }
                //check if the password is correct
                //비밀번호를 hash 한후 데이터베이스에 있는 hash된 비번과 같은지 확인
                const passwordCorrect = await user.checkPassword(password); //여기의 user와 위의 const user와는 다름.. 전자는 entity 
                if (!passwordCorrect){
                    return{
                        ok:false,
                        error:"잘못된 비밀번호입니다",
                    };
                }
                const token = this.jwtService.sign(user.id);//이 module을 내 백엔드만으로 특정함
                return{
                    ok:true,
                    token,
                }
            }catch(error){
                return{
                    ok: false,
                    error,
                };
            }
            
        }
        async findById(id:number): Promise<User>{
            return this.users.findOne({id});
        }
    
        async editProfile(userId: number, {email, password}: EditProfileInput){
            const user = await this.users.findOne(userId);
            if(email){
                user.email = email
            }
            if(password){
                user.password = password
            }
            return this.users.save(user) //db에 entity 존재 유무 체크 안함
        }// save는 entity 없으면 생성함
    
        async deleteAccount(userId: number, {email, password, studentId}: DeleteAccountInput){
            const user = await this.users.findOne(userId);
            try{
                if(!user){ //user가 존재하지 않는다면
                return {
                    ok:false,
                    error: '없는 계정입니다',
                }
                }
                await this.users.delete(userId);
                return{
                    ok: true,
                }
            }catch(error){
                return{
                    ok: false,
                    error,
                };
            }
            
        }
    }
    ```

  * core.entity.ts

    ```typescript
    import { Field, ObjectType } from "@nestjs/graphql";
    import { CreateDateColumn, DeleteDateColumn, PrimaryGeneratedColumn, UpdateDateColumn } from "typeorm";
    
    @ObjectType()
    export class CoreEntity {
        @PrimaryGeneratedColumn()
        @Field(type=>Number) //graphQL type만들기
        id:number;
    
        @CreateDateColumn()
        @Field(type=>Date)
        createdAt:Date;
    
        @UpdateDateColumn()
        @Field(type=>Date)
        updatedAt:Date;
    
        @DeleteDateColumn()
        @Field(type=>Date)
        deletedAt: Date;
    
    }
    ```

    

