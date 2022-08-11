# clean-code-example

#### Origin Code 
```javascript
httpClient.interceptors.request.use((config: AxiosRequestConfig) => {
  const snakeParseConfig = { ...config }
  const token = AuthStorage.get()
  // eslint-disable-next-line no-param-reassign
  snakeParseConfig.headers.Authorization = `Bearer ${token}`
  if (snakeParseConfig.headers['Content-Type'] === 'multipart/form-data')
    return snakeParseConfig

  if (snakeParseConfig.params)
    snakeParseConfig.params = decamelizeKeys(config.params)
  if (snakeParseConfig.data) snakeParseConfig.data = decamelizeKeys(config.data)
  return snakeParseConfig
})
```

#### Refactored Code
```javascript
export const requestInterceptor = httpClient.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    /**
     * 익명 함수 이지만...
     * 역할 : 실제 request를 보내기전 interceptor 한 requestconfig를
     * 사용자 정의를 추가하여 requestconfig 를 반환한다.
     */
    const authorizedRequestConfig = getAuthorizedRequestConfig(config)
    if (isContentTypeFormData(authorizedRequestConfig))
      return authorizedRequestConfig
    return getDecamelizedRequestConfig(authorizedRequestConfig)
  },
)

/**
 * 역할 : decamelize 된 (snake case 로 변환된) requestconfig 를 반환한다.
 */
const getDecamelizedRequestConfig = (config: AxiosRequestConfig) => {
  const decamelizedRequestConfig = { ...config }
  decamelizedRequestConfig.params = config.params ?? decamelizeKeys(config.params)
  decamelizedRequestConfig.data = config.data ?? decamelizeKeys(config.data)
  return decamelizedRequestConfig
}

/**
 * 역할 : authorization header 가 추가된 (authorized) requestconfig 를 반환한다.
 */
const getAuthorizedRequestConfig = (config: AxiosRequestConfig) => {
  const authorizedRequestConfig = { ...config }
  authorizedRequestConfig.headers.Authorization = getBearerTypeAuthorizationField()
  return authorizedRequestConfig
}

/**
 * 역할 : bearer 타입의 authorization 필드 값을 반환한다.
 */
const getBearerTypeAuthorizationField = () => {
  return `Bearer ${AuthStorage.get()}`
}

/**
 * 역할 : config header의 content-type 이 form-data 타입인지 확인한다.
 */
const isContentTypeFormData = (config: AxiosRequestConfig) => {
  return config.headers['Content-Type'] === 'multipart/form-data'
}
```
