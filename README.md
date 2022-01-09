# GithubRepository

> RxSwift 를 이용하여 URL을 받고 tableView에 뿌려줌  
> RxSwift 연습을 위한 구현

## 구현 완성 화면

![simulator_screenshot_BC0AE353-2C68-4563-B821-5AE15C8CBBDE](https://user-images.githubusercontent.com/61230321/148521807-5c5b5b65-0676-46bb-a25b-146efb8245e0.png)

### RxSwift 구현 코드 리뷰

```swift
   Observable.from([organization])  
            .map{ organization -> URL in
                return URL(string:
                            "http://api.github.com/orgs/\(organization)/repos")!
            }
 ```
 - 함수의 전달 인자로 of를 사용하였고, Observable.from은 배열의 값을 보내기 때문에 [organization] 으로 입력
 - //내려온 organization을 map을 통해 URL로 만들어 줌


  
  

```swift
   .map{ url -> URLRequest in
                var request = URLRequest(url: url)
                request.httpMethod = "GET"
                return request
            }
  ```
  - 내려온 url을 URLReguest에 넣음
  - request.httpMethod를 통해 "GET"을 받아옴

```swift
  
   .flatMap{ request -> Observable<(response: HTTPURLResponse, data: Data)> in
       return URLSession.shared.rx.response(request: request)
   }
```
- flatMap을 통해 return 받는 Observable 안의 내용을 받음
- URLSession.shared.rx.response(request: request) 로 위에서 받아온 request 활용

```swift
            .filter{ response,_ in
                return 200..<300 ~= response.statusCode
            }
            .map{_, data -> [[String: Any]] in
                guard let json = try? JSONSerialization.jsonObject(with: data, options: []),
                      let result = json as? [[String: Any]] else {
                    return []
                }
                return result
            }
```            
- 위에서 받아온 response가 통신 성공값인 것만 필터링해서 내려보냄
- 필터링 되어 내려온 것 중의 data를 JsonParsing 함


```swift
            .filter{ result in
                result.count > 0
            }
            .map{ objects in
                return objects.compactMap{ dic -> Repository? in
                    guard let id = dic["id"] as? Int,
                          let name = dic["name"] as? String,
                          let description = dic["description"] as? String,
                          let stargazersCount = dic["stargazers_count"] as? Int,
                          let language = dic["language"] as? String else {
                        return nil
                    }
                    return Repository(id: id, name: name, description: description, stargazersCount: stargazersCount, language: language)
                }
            }
 ``` 
 - filter를 통해 빈 배열은 걸러서 내려보냄.
 - map으로 내려온 result(=objects)를 compactMap을 통해 필요한 값들을 Rspository에 저장하고 리턴

```swift
            .subscribe(
                onNext: { [weak self] newRepositories in
                    self?.repositories.onNext(newRepositories)
                    
                    DispatchQueue.main.async {
                        self?.tableView.reloadData()
                        self?.refreshControl?.endRefreshing()
                    }
                }
            ).disposed(by: disposeBag)
```
- repositories라고 만들어 둔 behaviorSubject에 값을 저장함
