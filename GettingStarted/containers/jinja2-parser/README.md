# Jinja2 live parser

Source: https://github.com/qn7o/jinja2-live-parser

### Dockerfile

Build it:

    docker build -t jinja/j2parser .
    docker run -d -p 5000:5000 jinja/j2parser

## Usage

You are all set, go to `http://localhost:5000/` and have fun.  
You can add any custom filter you'd like in `filters.py`.  Just make sure the function's name starts with `filter_`.


## Preview

![preview](http://i.imgur.com/T65xjAf.png)
