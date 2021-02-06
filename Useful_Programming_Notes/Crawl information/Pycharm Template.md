# Pycharm template

## Predefined template variables﻿

The following predefined variables can be used in file templates:

| Variable              | Description                                                  |
| --------------------- | :----------------------------------------------------------- |
| `${DATE}`             | Current system date                                          |
| `${DAY}`              | Current day of the month                                     |
| `${DS}`               | Dollar sign `$`. This variable is used to escape the dollar character, so that it is not treated as a prefix of a template variable. |
| `${FILE_NAME}`        | Name of the new file.                                        |
| `${HOUR}`             | Current hour                                                 |
| `${MINUTE}`           | Current minute                                               |
| `${MONTH}`            | Current month                                                |
| `${MONTH_NAME_FULL}`  | Full name of the current month (January, February, and so on) |
| `${MONTH_NAME_SHORT}` | First three letters of the current month name (Jan, Feb, and so on) |
| `${NAME}`             | Name of the new entity (file, class, interface, and so on)   |
| `${PRODUCT_NAME}`     | Name of the IDE (for example, PyCharm)                       |
| `${PROJECT_NAME}`     | Name of the current project                                  |
| `${TIME}`             | Current system time                                          |
| `${USER}`             | Login name of the current user                               |
| `${YEAR}`             | Current year                                                 |

## Custom template variables﻿

​             Besides predefined template variables, it is possible to  specify custom variables.             If necessary, you can define the  values of custom variables right in the template using the `#set` directive.         

For example, if you want to use your full name instead of your login name defined through the predefined variable `${USER}`, use the following construct:

```none
#set( $MyName = "John Smith" )
```



Copied!

If the value of a variable is not defined in the template, PyCharm will ask you to specify it when the template is applied.





### sample includes file 

```python
# Created by ${USER}, Date ${DATE}
# File name ${FILE_NAME}
# ----------------------------------
# //////////////////  Description  ///////////////
# 
# 
#

```





### sample template

```python
#parse ("header.py")

```

note that the template file must add Enter after the template 