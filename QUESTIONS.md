### ant design pro 遇到的问题
* upload组件上传file文件（excel）
[参考地址](https://blog.csdn.net/a2469025185/article/details/82770762)
* 首先是调用 request方法

```

import request from '../utils/request';

export async function createNotice(params){
  const path = "https://xxxxxxx"
  return request(path, {
    method: 'fileUpload', 
    body: {
      ...params
    },
  });
}

```
* 在原有的antd pro 的request 方法中进行一点改变

```
import fetch from 'dva/fetch';
import { notification } from 'antd';
import { routerRedux } from 'dva/router';
import store from '../index';
import FormData from 'form-data'


const codeMessage = {
  200: '服务器成功返回请求的数据。',
  201: '新建或修改数据成功。',
  202: '一个请求已经进入后台排队（异步任务）。',
  204: '删除数据成功。',
  400: '发出的请求有错误，服务器没有进行新建或修改数据的操作。',
  401: '用户没有权限（令牌、用户名、密码错误）。',
  403: '用户得到授权，但是访问是被禁止的。',
  404: '发出的请求针对的是不存在的记录，服务器没有进行操作。',
  406: '请求的格式不可得。',
  410: '请求的资源被永久删除，且不会再得到的。',
  422: '当创建一个对象时，发生一个验证错误。',
  500: '服务器发生错误，请检查服务器。',
  502: '网关错误。',
  503: '服务不可用，服务器暂时过载或维护。',
  504: '网关超时。',
};
function checkStatus(response) {
  if (response.status >= 200 && response.status < 300) {
    return response;
  }
  const errortext = codeMessage[response.status] || response.statusText;
  notification.error({
    message: `请求错误 ${response.status}: ${response.url}`,
    description: errortext,
  });
  const error = new Error(errortext);
  error.name = response.status;
  error.response = response;
  throw error;
}

/**
 * Requests a URL, returning a promise.
 *
 * @param  {string} url       The URL we want to request
 * @param  {object} [options] The options we want to pass to "fetch"
 * @return {object}           An object containing either "data" or "err"
 */
export default function request(url, options) {
  const defaultOptions = {
    credentials: 'include',
  };
  let newOptions = { ...defaultOptions, ...options };
  if (
    newOptions.method === 'POST' ||
    newOptions.method === 'PUT' ||
    newOptions.method === 'DELETE'
  ) {
    if (!(newOptions.body instanceof FormData)) {
      newOptions.headers = {
        Accept: 'application/json',
        'Content-Type': 'application/json; charset=utf-8',
        ...newOptions.headers,
      };
      newOptions.body = JSON.stringify(newOptions.body);
    } else {
      // newOptions.body is FormData
      newOptions.headers = {
        Accept: 'application/json',
        ...newOptions.headers,
      };
    }
  }else if (newOptions.method === 'fileUpload') {//这部分为上传文件
	    newOptions.method = 'POST'
	    const dataParament = newOptions.body;
	    console.log("dataParament ==",dataParament )//你可以在这里查看你要传的文件对象
	    let filedata = new FormData();
	    if(newOptions.body.files){ 
	    //这里可以包装多文件 
	      for(let i = 0 ;i<dataParament.files.fileList.length;i++){
	       //dataParament.files.fileList[i].originFileObj 这个对象是我观察 antd的Upload组件发现的里面的originFileObj 对象就是file对象
	        filedata.append('files',dataParament.files.fileList[i].originFileObj)
	      }
	    }
	    for (let item in dataParament) {
	      if(item != 'files' && dataParament[item]){ 
		//除了文件之外的 其他参数 用这个循环加到filedata中
	        filedata.append(item,dataParament[item])
	      }
	    }
	    newOptions.body = filedata
  }
  return fetch(url, newOptions)
    .then(checkStatus)
    .then(response => {
      if (newOptions.method === 'DELETE' || response.status === 204) {
        return response.text();
      }
      return response.json();
    })
    .catch(e => {
      const { dispatch } = store;
      const status = e.name;
      if (status === 401) {
        dispatch({
          type: 'login/logout',
        });
        return;
      }
      if (status === 403) {
        dispatch(routerRedux.push('/exception/403'));
        return;
      }
      if (status <= 504 && status >= 500) {
        dispatch(routerRedux.push('/exception/500'));
        return;
      }
      if (status >= 404 && status < 422) {
        dispatch(routerRedux.push('/exception/404'));
      }
    });
}
```