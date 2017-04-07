# Promise规范的简单实现

``` javascript
function start () {
        var p = new Promise();
        console.log('start');
        p.resolve();
        return p;
    }

    function test1() {
        console.log(1);
    }

    function sleep1() {
        var p = new Promise();

        setTimeout( function(){
            console.log('sleep1');
            p.resolve();
        }, 200);

        return p;
    }

    function test2() {
        console.log(2);
    }

    function sleep2() {
        var p = new Promise();

        setTimeout( function(){
            console.log('sleep2');
            p.resolve();
        }, 300);

        return p;
    }

    function test3() {
        console.log(3);
        var x = 1;
        for(var i=0;i<10;i++){
            setTimeout(function(){
                x++;
                console.log(x)
            },30)
        }
    }
    function sleep3() {
        var p = new Promise();
        setTimeout( function(){
            console.log('sleep3');
            p.resolve();
        }, 400);
        return p;
        
    }

 start().then( test1, null )
        .then( sleep1(), null )
        .then( test2, null )
        .then( sleep2(), null )
        .then( test3, null )
        .then( sleep3(), null );

```