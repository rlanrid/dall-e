# VITE REACT로 AI 이미지 생성 사이트 만들기

## 🔧초기세팅   
`npm create vite@latest ./`   
`npm install -D tailwindcss postcss autoprefixer`   
`npx tailwindcss init -p`   
`npm install file-saver`   
`npm install react-router-dom`      

## 🧾개념 정리   
- React   
React는 페이스북에서 개발한 오픈 소스 자바스크립트 라이브러리로, 사용자 인터페이스를 만들기 위해 사용됩니다.

- Vite   
Vite는 Evan You가 만든 빠르고 간단한 웹 개발 빌드 도구입니다. Vite는 기본적으로 Vue.js 애플리케이션을 위해 설계되었지만,   
최근에는 React, Preact, Svelte 등 다양한 프레임워크와 라이브러리를 지원하고 있습니다.   

- File-saver  
JavaScript 라이브러리로, 웹 애플리케이션에서 클라이언트 측에서 파일을 생성하고 다운로드하는 데 사용됩니다.

- Tailwind CSS   
Tailwind CSS는 사용자 인터페이스를 디자인하고 개발하는 데 사용되는 현대적이고 인기 있는 CSS 프레임워크입니다.   

- MongoDB   
MongoDB는 NoSQL 데이터베이스로, 문서 지향(Document-Oriented) 데이터베이스 시스템입니다.   
관계형 데이터베이스와 달리 데이터를 JSON과 유사한 BSON 형식의 문서로 저장하며, 이러한 문서는 컬렉션(Collection) 안에 저장됩니다.   

- Cloudinary   
Cloudinary는 클라우드 기반의 미디어 관리 플랫폼으로, 이미지 및 비디오 관련 작업을 수행하는 데 사용됩니다.   

- useEffect   
React의 useEffect는 함수 컴포넌트에서 부수 효과(side effects)를 처리하기 위한 Hook입니다. 부수 효과란 컴포넌트 외부와의 상호작용을 의미하며,   
주로 데이터 가져오기, 구독 설정, 이벤트 처리 등이 해당됩니다. useEffect를 사용하면 컴포넌트의 생명주기에 따라 특정 동작을 수행할 수 있습니다.

## 🔍주요 기능   
[이미지 생성]   
   
Openai API를 이용해 프롬프트를 입력하면 이미지를 생성하는 기능을 구현했습니다.   

```js
router.route('/').post(async (req, res) => {
    try {
        const { prompt } = req.body;

        const aiResponse = await openai.images.generate({
            prompt,
            n: 1,
            size: '1024x1024',
            response_format: 'b64_json',
        });

        const image = aiResponse.data[0].b64_json;
        res.status(200).json({ photo: image });
    } catch (error) {
        console.error(error);
        res.status(500).send(error?.response.data.error.message || 'Something went wrong');
    }
});
```   

클라이언트에서 전송된 prompt를 받아 openai 문서를 참조해 이미지를 생성합니다.   

[이미지 공유]   

```js
router.route('/').post(async (req, res) => {
    try {
        const { name, prompt, photo } = req.body;
        const photoUrl = await cloudinary.uploader.upload(photo);

        const newPost = await Post.create({
            name,
            prompt,
            photo: photoUrl.url,
        })

        res.status(201).json({ success: true, data: newPost });
    } catch (error) {
        res.status(500).json({ success: false, message: error });
    }
});
```   

이미지 공유 버튼을 클릭하면, 클라이언트로부터 받아온 이름, 프롬프트, 사진 정보를 받아오고, 데이터베이스에 추가합니다.   
사진은 cloudinary를 이용해 업로드 한 뒤, 사진 주소를 저장합니다.

[커뮤니티]   


```js
useEffect(() => {
    const fetchPosts = async () => {
        setLoading(true);

        try {
            const response = await fetch('https://dall-e-gpw2.onrender.com/api/v1/post', {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json',
                },
            })

            if (response.ok) {
                const result = await response.json();

                setAllPosts(result.data.reverse());
            }
        } catch (error) {
            alert(error);
        } finally {
            setLoading(false);
        }
    }

    fetchPosts();
}, []);
```

useEffect 훅을 사용해 렌더링 될때마다 게시글들을 불러옵니다.   

```js
router.route('/').get(async (req, res) => {
    try {
        const posts = await Post.find({});

        res.status(200).json({ success: true, data: posts });
    } catch (error) {
        res.status(500).json({ success: false, message: error });
    }
});
```   

데이터베이스에서 모든 게시글들을 찾아 클라이언트에 보내줍니다.   

## 😋트러블슈팅   
<details>
    <summary>
    error while updating dependencies:
    Error: ENOENT: no such file or directory,
    </summary>

    - 문제 원인
    버전 문제

    - 문제 해결
    버전을 이전 버전으로 설치
</details>   

## 📎사이트   
API - [openapi](https://openai.com/product)   
데이터베이스 - [MongoDB](https://www.mongodb.com/)   
배포 - [Vercel](https://vercel.com/), [Render](https://render.com/)   
데이터베이스 - [MongoDB](https://www.mongodb.com/)   

## 📘스택   
<div>
  <a href="#"><img alt="React" src="https://img.shields.io/badge/React-61DAFB?style=flat&logo=React&logoColor=white"></a>
  <a href="#"><img alt="Tailwind CSS" src="https://img.shields.io/badge/Tailwind CSS-06B6D4?logo=Tailwind CSS&logoColor=white"></a>
  <a href="#"><img alt="JavaScript" src="https://img.shields.io/badge/JavaScript-F7DF1E?logo=JavaScript&logoColor=white"></a>
  <a href="#"><img alt="MongoDB" src="https://img.shields.io/badge/MongoDB-47A248?logo=MongoDB&logoColor=white"></a>
  <a href="#"><img alt="OpenAI" src="https://img.shields.io/badge/OpenAI-412991?logo=OpenAI&logoColor=white"></a>
</div>