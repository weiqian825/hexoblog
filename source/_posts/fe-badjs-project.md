---
title: fe-badjs-project
date: 2018-08-16 15:31:13
tags:
---
页面收集badjs核心是重写window.onerror事件，整理里面的错误信息，上报到接口。
```
    window.onerror = function(msg, url, line, col, error) {
        var errStr, win = window;
        function obj2str(obj) {
            var t, sw;
            if (typeof(obj) == 'object') {
                if (obj === null) {
                    return 'null';
                }

                if (window.JSON && window.JSON.stringify) {
                    return JSON.stringify(obj);
                }

                sw = isArray(obj);
                t = [];
                for (var i in obj) {
                    t.push((sw ? "" : ("\"" + i + "\":")) + obj2str(obj[i]));
                }
                t = t.join();
                return sw ? ("[" + t + "]") : ("{" + t + "}");
            } else if (typeof(obj) == 'undefined') {
                return 'undefined';
            } else if (typeof(obj) == 'number' || typeof(obj) == 'function') {
                return obj.toString();
            }
            return !obj ? "\"\"" : ("\"" + obj + "\"");
        }

        function isArray(source) {
            return '[object Array]' == Object.prototype.toString.call(source);

        }

        col = col || (win.event && win.event.errorCharacter) || 0;
        if (!!error && !!error.stack) {
            //如果浏览器有堆栈信息
            //直接使用
            msg = error.stack.toString();
        } else if (!!arguments.callee) {
            //尝试通过callee拿堆栈信息
            var ext = [msg];
            var f = arguments.callee.caller,
                c = 3;
            //这里只拿三层堆栈信息
            while (f && (--c > 0)) {
                ext.push(f.toString());
                if (f === f.caller) {
                    break; //如果有环
                }
                f = f.caller;
            }
            ext = ext.join(",");
            msg = ext;
        }
        errStr = obj2str(msg) + (url ? ";URL:" + url : "") + (line ? ";Line:" + line : "") + (col ? ";Column:" + col : "");

        if (win._last_err_msg) {
            if (win._last_err_msg.indexOf(msg) > -1) return;
            win._last_err_msg += "|" + msg
        } else {
            win._last_err_msg = msg;
        }
        console.log('the error log need report to cgi ',errStr)
     
        return false;
    };
```
搭建一个nodejs服务，记录上报的badjs错误信息(mongodb+node)

