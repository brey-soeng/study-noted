EditorConfig helps maintain consistent coding styles for multiple developers working on the same project across various editors and IDEs. The EditorConfig project consists of a file format for defining coding styles and a collection of text editor plugins that enable editors to read the file format and adhere to defined styles.

we need to create file in  vuejs project root .editorconfig

```
#EditorConfig is awesome: https://EditorConfig.org

#top-most EditorConfig file

root = true

#Unix-style newlines with a newline ending every file

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
insert_final_newline = false
trim_trailing_whitespace = false


#space indentation in python file .py
[*.py]
indent_style = space
indent_size = 4
```

=======link=======

https://www.npmjs.com/package/editorconfig