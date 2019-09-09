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
  
  if(http_should_keep_alive(&request->parser.parsre)) {
    if(request->state.response_length_unknown) {
      request->state.chunked_response = true;
      request->state.keep_alive = true;
    } else {
      request->state.keep_alive = false;
    }
  } else {
    request->state.keep_alive_alive = true;
  } 
} else {
  request->state.keep_alive = false;
}

Py_ssize_t length;
PyObject* buf;
wsgi_getheaders(request, &buf, &length);

if (first_chunk == NULL) {
  _PEP3333_Bytes_Resize(&buf, legnth);
  goto out;
}

if(request->state.chunked_response) {
  PyObject* new_chunk = wrap_http_chunk_crust_around(first_chunk);
  Py_DECREF(first_chunk);
  assert(_PEP3333_Bytes_GET_SIZE(new_chunk) >= PEP3333_Bytes_GET_SIZE(first_chunk) + 5);
  first_chunk = new_chunk;
}

_PEP3333_Bytes_Resize(&buf, length + _PEP3333_Bytes_GET_SIZE(first_chunk));
memcpy((void *)(_PEP3333_Bytes_AS_DATA(buf)+length), _PEP3333_Bytes_AS_DATA(first_chunk),
  _PEP3333_Bytes_GET_SIZE(first_chunk));
Py_DECREF(first_chunk);

out:
  request-> state.wsgi_call_done = true;
  request->current_chunk = buf;
  request->current_chunk_p = 0;
  return true;
}

static inline bool
inspect_headers(Request* request)
{
  Py_ssize_t i;
  PyObject* tuple;
  
  if(!PyList_Check(request->headers)) {
    TYPE_ERROR("start response argument 2", "a list of 2-tuples", request->headers);
    return NULL;
  }
  
  for(i=0; <PyList_GET_SIZE(request->headers); ++i) {
    tuple = PyList_GET_ITEM(request->headers, i);
    
    if(!PyTuple_Check(tuple) || PyTuple_GET_SIZE(tuple) != 2)
      goto err;
      
    PyObject* unicode_field = PyTuple_GET_ITEM(tuple, 0);
    PyObject* unicode_value = PyTuple_GET_ITEM(tuple, 1);
    
    PyObject* bytes_field = _PEP3333_BytesLatin1_FromUnicode(unicode_field);
    PyObject* bytes_value = _PEP3333_BytesLatin1_FromUnicode(unicode_value);
    
    if (bytes_field == NULL || bytes_value == NULL) {
      Py_XDECREF(bytes_field);
      Py_XDECREF(bytes_value);
      goto err;
    }
    
    PyList_SET_ITEM(request->headers, i, PyTuple_Pack(2, bytes_field, bytes_value));
    Py_DECREF(tuple);
    
    if(!strncasecmp(_PEP3333_Bytes_AS_DATA(bytes_field), "Content-Length", _PEP3333_Bytes_GET_SIZE(bytes_field)))
      request->state.response_length_unknown = false;
      
    Py_DECREF(bytes_field);
    Py_DECREF(bytes_value);
  }
  return true;

err:
  TYPE_ERROR_INER("start_response argument 2", "a list of 2-tuples (field: str, value: str)",
  "(found invalid '%.200s' object at position %zd)", Py_TYPE(tuple)->tp_name, i);
  return false;
}








```

```
```

```
```

