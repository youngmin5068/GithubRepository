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
            .map{ url -> URLRequest in
                var request = URLRequest(url: url)
                request.httpMethod = "GET"
                return request
            }
            .flatMap{ request -> Observable<(response: HTTPURLResponse, data: Data)> in
                return URLSession.shared.rx.response(request: request)
            }
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
