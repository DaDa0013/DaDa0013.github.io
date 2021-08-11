---
title: "[project Oechung]로그인 frontend 구현"
categories:
  - Frontend
  - Project
tags:
  - React
  - Html
  - Css
  - JS
toc: true
toc_sticky: true
excerpt: "frontend"
---

# 로그인 페이지

## 구현한 기능(Html)

* 회원가입 페이지로 넘어가는 링크

* 아이디 찾기 & 비밀번호 찾기 링크

* 아이디와 비밀번호를 받아 콘솔창에 출력하기

* 비밀번호는 입력할때 암호 형식으로 받기

  ```html
  import React, { Component } from 'react';
  import './Button.css';
  class Loginform extends Component {
    
  
    constructor(props) {
      super(props);
      this.state = { value: '' };
      this.appChange = this.appChange.bind(this);
      this.appClick = this.appClick.bind(this);
    }
  
    // goToRegister = () => {
    //   this.props.history.push('/register');
    // }
  
    appChange = (e) =>{
      this.setState({
        [e.target.name]:e.target.value
      });//input value 변경 ==> onChange
    }
    appClick = (e) =>{
      console.log(`id는:${this.state.ID}\n pw는:${this.state.PW}`); 
      alert('로그인성공!');
      e.preventDefault();
      //this.goToMain();
      //로그인 성공시 메인페이지로 이동하도록
    }
    appKeyPress = (e) => {
      if (e.key === 'Enter'){
        this.appClick();
      }
    }
    render(){
      const { ID, PW } = this.state;
      const { appChange, appClick } = this;
      return (
        <body className="background">
          <img className="image" src={require("../img/logo.png").default}></img>
          <div className="bigbox">
            <form action="/input" method="post">
              <h1>LOGIN</h1>
              <p><input className="inputbox" type="text" name="ID" placeholder="ID" value={ID} onChange={appChange}></input></p>
              <p><input className="inputbox" type="password" name="PW" placeholder="PASSWORD" value={PW} onChange={appChange}></input></p>
              <p><button className="button" onClick={ appClick }>Login</button></p>
              <div className="links">
                <a href="#" target="_blank">회원가입페이지로</a>
              </div>
              <div className="link2">
                <a  href="#" target="_blank">아이디찾기</a>
              </div>  
              <div className="link2">
              <a href="#" target="_blank">비밀번호찾기</a>
              </div>
            </form>
          </div>
        </body>
      );
    }
  }
  export default Loginform;
  ```



## 구현한 기능(Css)

* 배경설정

  (위치를 맞추느라 힘들었음)

* 로그인 버튼 스타일

* 로그인 창 스타일

  ```css
  
    h1 {
      position: relative;
      top:20px;
      bottom: 450x;
      left: 85px;
      font-size: 50px;
      font-family: serif;
      text-align: left;
      font-weight: bolder;
      margin-bottom: 70px;
      margin-top:0px;
    }
    p{
      text-align: center;
    }
    .background{
      background-image: url("../img/background.png");
      background-size: cover;
      width: 100%;
      height: 100%;
      position:absolute;
      top:0;
      left:0;
      /* background-color:skyblue;
      background-size: cover;
      position:absolute;
      top:0;
      left:0;
      width: 100%;
      height:100%; */
    }
    .image{
      width: 200px;
      height:50px;
      position: relative;
      top: 170px;
      left: 400px;
    }
    .bigbox{
      background-color: white;
      padding: 20px;
      /* padding-bottom: 150px; */
      padding-top: 70px;
      padding-bottom: 70px;
      padding-left: 50px;
      padding-right: 100px;
      margin-top: 100px;
      margin-bottom: 100px;
      margin-left: auto;
      margin-right: auto;
      border-radius: 20px;
      border-style: groove;
      width: 300px;
      height: 400px;
    }
    .inputbox{
      position: relative;
      left: 0px;
      top: 20px;
      bottom: 10px;
      width: 200px;
      height: 30px;
    }
    .button{
      position: relative;
      left: 170px;
      bottom: 80px;
      color: white;
      font-family: serif;
      font-weight: 700;
      font-size: 20px;
      margin-bottom: 30px;
      width: 100px;
      height: 80px;
      background-color: rgb(8, 80, 139);
      /* border-radius: 5px; */
      border: none;
    }
    .links{
      position: relative;
      bottom: 10px;
      text-align: center;
      color:lightslategrey;
      font-size: 12px;
    }
    .link2{
      position: relative;
      bottom: 20px;
      left: 90px;
      padding:80;
      text-align: center;
      color:black;
      font-size: 12px;
      font-weight: 700;
      margin-left: auto;
      margin-right: auto;
    }
  © 2021 GitHub, Inc.
  Terms
  Privacy
  Security
  Status
  Docs
  Contact GitHub
  Pricing
  API
  Training
  Blog
  About
  Loading complete
  ```

  

