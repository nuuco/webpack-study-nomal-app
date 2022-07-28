# 웹팩(Webpack) 번들링 연습
👉 [리액트 앱 웹팩 번들링 연습 바로가기](https://github.com/nuuco/webpack-study-react-app)

## 📌 Webpack 을 적용해보자

1. **package.json 만들기**
    - 우선 다음 명령어로 기본 설정된 package.json 파일을 만들어준다.
    
    ```bash
    $ npm init -y
    ```
                
<br/>    

2. **webpack 설치하기**
    - 웹팩을 사용하려면 webpack 과 webpack-cli 를 설치해야 한다.   
        ※ 개발 환경에서만 필요한 모듈이므로 -D (또는 —save-dev) 로 설치
        
    
    ```bash
    $ npm install -D webpack webpack-cli
    ```
               
<br/>    
 
3. **webpack.config.js 파일 만들기**
    - 웹팩 설정 파일을 만들어준다. 번들 파일을 만들기 위해서는 기본적으로 entry, output 두 속성이 필요하다.
        - entry : 번들링 시작 파일 위치
        - output : 번들링 결과물
    - 이때 파일 경로를 쉽게 작성하기 위해 path 모듈을 주로 사용한다.
        
        ※ path.resolve(’경로1’, ‘경로2’…) 는 인자로 받은 경로를 합쳐준다. → 경로1/경로2
        
    
    ```jsx
    //webpack.config.js
    const path = require('path');
    
    module.exports = {
      entry: './src/index.js',
      output: {
        path: path.resolve(__dirname, 'dist'), // './dist'의 절대 경로를 리턴
        filename: 'app.bundle.js',
      },
    };
    ```
             
<br/>    
   
4. **webpack 스크립트 세팅**
    - `npx webpack` 명령어로 웹팩을 실행시킬 수 있다. 이러면 entry 파일에서부터 시작해 번들링한 결과를 output 에 지정한 파일로 만들어 준다.
    - 다른 개발자와 협업할 때 용이하게 하기 위해 스트립트를 만들어 주는 게 좋다.
    
    ```jsx
    //package.json
    "scripts": {
    	"build": "webpack", // here
        "test": "echo \"Error: no test specified\" && exit 1"
      },
    ```
               
<br/>    
 
5. **의존성 모듈 js 파일에 넣어주기**
    - js 파일에 **require** 로 필요한 파일을 넣어준다. (외부파일, 이미지, css 등등…)
    - 필요한 모듈을 js 파일 안에서  require 로 가져와야 웹팩이 필요한 파일로 인식해 번들링할때 같이 묶어준다. (js 파일이라면 module.exports 로 내보내주는 것 잊지 말자.)
        
        ```jsx
        //./src/script.js
        require('./style.css');
        const { agoraStatesDiscussions } = require('./data.js');
        ...
        
        //./src/data.js
        module.exports.agoraStatesDiscussions = {...};
        ```
                 
<br/>       
    
6. **로더(loader) 로 다른 유형 파일 전처리 해주기**
    - 웹팩은 기본적으로 js 와 JSON 만 이해하기 때문에, 다른 파일을 넣어준 경우 로더(loader) 로 전처리 해줘야한다.
        
        ※ css 파일을 js 파일에 넣어준 경우, 웹팩이 파일을 이해하지 못해 에러 발생
        
    - 사용하는 파일(모듈) 종류에 따라 적합한 로더를 설치한 뒤, webpack.config.js 파일의 module 속성 안에 작성해준다.
    - 💻 예를 들어, css 파일을 번들파일(js)에 적용하고 싶다면
        1. css-loader 와 style-loader 를 설치한다.
            
            ※ css-loader 는 css 파일을 json 으로 변환시켜주고, style-loader 는 이 json 을 style 태그 안에 넣고 html 문서에 삽입해준다. 즉, style-loader 가 없으면 css 가 반영되지 않는다.
            
            ```bash
            $ npm i -D css-loader style-loader
            ```
            
        2. webpack 설정 파일에 다음과 같이 추가한다.
            
            ```jsx
            module.exports = {
            	...
              module: {
                rules: [
                  {
                    test: /\.css$/,
                    use: ["style-loader", "css-loader"],
            				//이때, 오른쪽부터 로더가 적용되므로 css-loader가 맨 오른쪽이어야한다.
                    exclude: /node_modules/,
                  }
                ],
              },
            };
            ```
            
        3. 웹팩을 실행시키면 이렇게 번들 파일(js)에 css 가 문자열로 들어가 있고, 실행한 html 문서에 `<style>` 태그가 삽입되어 있다.(실제 파일엔 없고, 실행시 DOM 에 삽입되어 있다.)
                
<br/>    
        
7. **플러그인(plugin) 사용하기**
    - 플러그인을 사용해서 다양한 작업을 편리하게 할 수 있다.
    - 💻 웹팩은 기본적으로 js 파일하나만 번들 파일로 만들기에 html 문서는 번들 폴더에 따로 생성해줘야했다. 하지만 `html-webpack-plugin` 을 사용하면 html 도 번들 폴더에 자동 생성해준다.
        1. 플러그인 설치
            
            ```bash
            $ npm i -D html-webpack-plugin
            ```
            
        2. webpack 설정 파일에 플러그인을 불러오고, plugins 속성에 다음과 같이 추가한다.
            
            ```jsx
            const path = require('path');
            //설치한 플러그인은 반드시 불러와야한다.
            const HtmlWebpackPlugin = require('html-webpack-plugin');
            
            module.exports = {
              ...
              plugins: [new HtmlWebpackPlugin({
                template: path.resolve(__dirname, "src", "index.html")
                //번들 폴더에 생성해준 기존 html 문서 위치
              })]
            };
            ```
            
        3. 웹팩을 실행하면 번들 폴더에 html 문서가 생기고 자동으로 번들링된 js 문서가 script 태그로 들어가 있다.
            
<br/>    

8. **추가 설정하기 - target, mode, output.clean 등**
    - target: ["web", "es5"] :    
    web 용 es5 문법으로 컴파일(번들링) 하겠다. → ES6를 지원하지 않는 브라우저에서 실행 가능 → **브라우저 호환성** ⬆️
    - mode: "production" :    
    배포 모드로 설정 → 코드가 축약화 되어 있다. 개발모드는 “development” → 코드 읽기 & 에러 찾기 수월 
    - output.clean: true :    
    새 번들링 시, 기존에 있던 번들 파일을 지운다.
    
    ```jsx
    //webpack.config.js
    
    const path = require('path');
    const HtmlWebpackPlugin = require('html-webpack-plugin')
    
    module.exports = {
      target: ["web", "es5"],
      mode: "production",
      entry: './src/script.js',
      output: {
        path: path.resolve(__dirname, 'dist'), // './dist'의 절대 경로를 리턴
        filename: 'app.bundle.js',
        clean: true,
      },
      module: {
        rules: [
          {
            test: /\.css$/,
            use: ["style-loader", "css-loader"],
            exclude: /node_modules/,
          }
        ],
      },
      plugins: [new HtmlWebpackPlugin({
        template: path.resolve(__dirname, "src", "index.html")
      })]
    };
    ```