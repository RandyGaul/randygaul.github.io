---
layout: post
title:  "Error Codes and Error Handling"
date:   2019-04-26 15:28:52 -0700
categories: api-design
---
There is a longstanding and ongoing debate in the industry on whether exceptions or error codes are superior when it comes to handling and report errors. I will share my opinion.

Error codes are better.

Here are some reasons to bias against exceptions.

* They destroy code flow.
* Users end up try-catching exceptions everywhere, and it devolves to more or less error codes.
* Handling an error in a centralized location is rarely useful, which is often the motivation for exceptions.

The simplest way to implement error codes is to return an integer value representing success or failure. Sometimes this integer gets upgraded to an enumeration for a little more descriptive flavor. Sometimes the codes or the enumeration are mapped to “details”. The details are the human readable description of a specific error. Sometimes the details live on a webpage, and sometimes they live in a big switch statement somewhere in the code.

After talking with a friend, they suggested creating an error struct containing both an error code along with a details string. Here’s an example of what I came up with.

{% highlight cpp %}
#define ERROR_CODE_FAILURE -1
#define ERROR_CODE_SUCCESS 0

struct error_t
{
	int code;
	const char* details;

	inline int is_error() const { return code == ERROR_CODE_FAILURE; }
};

inline error_t error_make(int code, const char* details) { error_t error; error.code = code; error.details = details; return error; }
inline error_t error_failure(const char* details) { return error_make(ERROR_CODE_FAILURE, details); }
inline error_t error_success() { return error_make(ERROR_CODE_SUCCESS, NULL); }
{% endhighlight %}

Functions can now return an `error_t` struct by value whenever reporting errors. Users can choose to respond or ignore to codes as they see fit. There is no need for thread local storage, exceptions, or any other fancy features. The strings themselves should be stored as a string literal, and I would recommend *not* localizing them for simplicity’s sake.

User code now will look mostly like this excerpt.

{% highlight cpp %}
error_t err = do_something(params);
if (err.is_error()) {
	handle_error(err.details);
}
{% endhighlight %}

Once error codes like this are setup any kind of long jump logic can be added as a separate feature, but the basic error codes themselves do not need complicated jump functionality built-in.
