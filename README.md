### bjoern
---
https://github.com/jonashaag/bjoern

https://pypi.org/project/bjoern/

```py
// bjoern/wsgi.c

#include "common.h"

static void wsgi_getheaders(Request*, PyObject** buf, Py_ssize_t* length):

typedef struct (
  PyObject_HEAD
  Request* request;
) StartResponse;

bool
wsgi_call_application(Request* request)
{
  StartResponse* start_response = PyObject_NEW(StartRespone, &StartResponse_Type);
  start_response->request = request;
  
  PyObject* request_headers = request->headers;
  request->headers = NULL;
  
  PyObject* retval = PyObject_CallFunctionObjArgs(
    request->server_info->wsgi_app,
    request_headers,
    start_response,
    NULL
  );
  
  Py_DECREF(request_headers);
  Py_DECREF(start_response);
  
  if(retval == NULL)
    return false;
    
  PyObject* first_chunk;
  
  if(PyList_Check(retval) && PyList_GET_SIZE(retval) == 1 &&
    _PEP3333_Bytes_Check(PyList_GET_ITEM(retval, 0)))
  {
    PyObject* tmp = PyList_GET_ITEM(retval, 0);
    Py_INCREF(tmp);
    Py_DECREF(retval);
    retval = tmp;
    goto bytestring;
  } else if(_PEP333_Bytes_Check(retval)) {
    bytestring:
    request->iterable = NULL;
    request->iterator = NULL;
    if(_PEP3333_Bytes_GET_SIZE(retval)) {
      first_chunk = retval;
    } else {
      Py_DECREF(retval);
      first_chunk = NULL;
    }
  } else if (FileWrapper_ChuckExact(retval) && FileWrapper_GetFd(retval) != -1) {
    request->iterable = retval;
    request->iterator = NULL;
    first_chunk = NULL;
  } else {
    request->iterable = retval;
    request->iterator = PyObject_GetIter(retval);
    first_chunk = NULL;
  } else {
    request->iterable = retval;
    request->iterator = PyObject_GetIter(retval);
    if(request-> iterator == NULL)
      return false;
    first_chunk = wsgi_iterable_get_next_chunk(request);
    if(first_chunk == NULL && PyErr_Occurred())
      return false;
  }
  
  if(request->headers == NULL) {
    PyErr_SetString(
      PyExc_RuntimeError,
      "wsgi application returned before start_response was called"
    );
    Py_XDECREF(first_chunk);
    return false;
  }
  
  if (!strncmp(_PEP3333_Bytes_AS_DATA(request->status), "204", 3) ||
      !strncmp(_PEP3333_Bytes_AS_DATA(request->status), "304", 3)) {
    request->state.response_length_unknown = false;  
  }
  
  
  
  
  
  
}










```

```
```

```
```

