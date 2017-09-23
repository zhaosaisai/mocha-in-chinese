## mocha中文文档

### 安装
使用[npm](https://npmjs.org/)全局安装：

```bash
$ npm install --global mocha
```
也可以作为项目的依赖进行安装：

```bash
$ npm install --save-dev mocha
```

> 安装Mocha >= v3.0.0，npm的版本应该>=v1.4.0。除此，确保使用Node.js的版本>=v0.10来运行Mocha

Mocha也可以使用[Bower](http://bower.io/)进行安装(`bower install mocha`)，也可以从[cdnjs](https://cdnjs.com/libraries/mocha)上获取。

### GEETING STARTED

```bash
$ npm install mocha
$ mkdir test
$ $EDITOR test/test.js # 或者使用你喜欢的编辑器
```

在编辑器中输入：

```javascript
var assert = require('assert')
describe('Array', function() {
    describe('#indexOf()', function() {
        it('should return -1 when the value is not present', function() {
            assert.equal(-1, [1, 2, 3].indexOf(4))
        })
    })
})
```
然后在终端中运行：

```bash
$ ./node_modules/mocha/bin/mocha

Array
    #indexOf()
      ✓ should return -1 when the value is not present


  1 passing (9ms)
```

在`package.json`中设置一个测试脚本：
 
```javascript
"scripts":{
    "test": "mocha"
}
```

然后运行：

```bash
$ npm test
```

### ASSERTIONS(断言)

Mocha允许你使用任意你喜欢的断言库，在上面的例子中，我们使用了Node.js内置的[assert](https://nodejs.org/api/assert.html)模块作为断言。如果能够抛出一个错误，它就能够运行。这意味着你能使用下面的这些仓库，比如：

* [should.js](https://github.com/shouldjs/should.js)
* [expect.js](https://github.com/LearnBoost/expect.js)
* [chai](http://chaijs.com/)
* [better-assert](https://github.com/visionmedia/better-assert)
* [unexpected](http://unexpected.js.org/)

### ASYNCHRONOUS CODE(异步代码)

使用mocha测试异步代码是再简单不过了。只需要在测试完成的时候调用一下回调函数即可。通过添加一个回调函数(通常命名为`done`)给`it()`方法，Mocha就会知道，它应该等这个函数被调用的时候才能完成测试。

```javascript
describe('User', function() {
    describe('#save()', function() {
        it('should save without error', function() {
            var user = new User('Luna')
            user.save(function(err) {
                if(err) done(err);
                else done()
            })
        })
    })
})
```

也可以让事情变得更简单，因为`done()`函数接收一个err，所以，我们可以直接按照下面的使用：

```javascript
describe('User', function() {
    describe('#save()', function() {
        it('should save without error', function(done) {
            var user = new User('Luna')
            user.save(done)
        })
    })
})
```

#### WORKING WITH PROMISES(使用promises)

同时，除了使用`done()`回调函数，你也可以返回一个[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。这种方式对于测试那些返回promies的方法是实用的。

```javascript
beforeEach(function() {
  return db.clear()
    .then(function() {
      return db.save([tobi, loki, jane]);
    });
});

describe('#find()', function() {
  it('respond with matching records', function() {
    return db.find({ type: 'User' }).should.eventually.have.length(3);
  });
});
```

> 后面的例子使用了[Chai as Promised](https://www.npmjs.com/package/chai-as-promised) 进行promise断言

在Mocha>=3.0.0版本中，返回一个promise的同时，调用了done函数。将会导致一个异常，下面是一个常见的错误：

```javascript
const assert = require('assert')

it('should complete this test', function (done) {
    return new Promise(function (resolve) {
        assert.ok(true)
        resolve()
    })
    .then(done)
})
```
这个测试会失败，错误信息为：` Error: Resolution method is overspecified. Specify a callback *or* return a Promise; not both.`
而比v3.0.0更老的版本中，调用done函数会被忽略。

### USING ASYNC / AWAIT
如果你的js运行环境支持[async/await](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/async_function)，你也可以像下面这样写异步测试。

```javascript
beforeEach(async function() {
  await db.clear();
  await db.save([tobi, loki, jane]);
});

describe('#find()', function() {
  it('responds with matching records', async function() {
    const users = await db.find({ type: 'User' });
    users.should.have.length(3);
  });
});
```

### SYNCHRONOUS CODE
当测试同步代码的时候，可以省略参数中的回调函数，Mocha会自动的测试下面的代码。

```javascrit
describe('Array', function () {
    describe('#indexOf()', function () {
        it('should return -1 when the value is not present', function() {
          [1,2,3].indexOf(5).should.equal(-1);
          [1,2,3].indexOf(0).should.equal(-1);
        });
    })
})
```

### ARROW FUNCTIONS
向Mocha传递箭头函数是不好的，由于this的词法作用域的问题，箭头函数是不能够访问mocha的上下文的。例如，由于箭头函数本身的机制，下面的代码会失败。

```javascript
describe('my suite', () => {
  it('my test', () => {
    // should set the timeout of this test to 1000 ms; instead will fail
    this.timeout(1000);
    assert.ok(true);
  });
});
```
如果你不需要使用mocha的上下文，可以使用箭头函数。然而，如果你以后需要使用这个上下文的话，重构会变得十分困难。

### HOOKS
鉴于默认使用BDD风格的接口，Mocha提供了一些钩子函数:`before()`,`after()`,`beforeEach()`和`afterEach()`。这些钩子函数可以用于设置测试的先决条件或者对测试进行清理。

```javascript
describe('hooks', function() {
    before(function() {
        // 在这个区块内的所有测试之前运行
    })
    after(function () {
        // 在这个区块内的所有测试之后运行
    })
    beforeEach(function () {
        // 在这个区块内的每个测试运行之前运行
    })
    afterEach(function () {
        // 在这个区块内的每个测试之后运行
    })
})
```

> 测试可以出现在before,after或者和你的钩子函数交替出现。钩子函数会按照它们被定义的顺序运行。一般就是，`before()(只运行一次)`->`beforeEach()`->`afterEach()`->`after()(只运行一次)`。

### DESCRIBING HOOKS

任何钩子函数在执行的时候都可以传递一个可选的描述信息，可以更容易地准确指出测试中的错误。如果钩子函数使用了命名的回调函数，则其名字会被作为默认的描述信息。

```javascript
beforeEach(function () {
    // beforeEach钩子函数(没有任何的描述信息)
})
beforeEach(function namedFn() {
    // beforeEach:namedFn会被当作描述信息
})
beforeEach('some description', function () {
    // beforeEach:some description(提供了描述信息)
})
```
### ASYNCHRONOUS HOOKS
所有的钩子(`before()`,`after()`,`beforeEach()`,`afterEach()`)可以是同步的也可以是异步的，其行为就像是普通的测试用例。例如，你希望在每个测试之前，向数据库中填充一些内容。

```javascript
describe('Connection', function() {
  var db = new Connection,
    tobi = new User('tobi'),
    loki = new User('loki'),
    jane = new User('jane');

  beforeEach(function(done) {
    db.clear(function(err) {
      if (err) return done(err);
      db.save([tobi, loki, jane], done);
    });
  });

  describe('#find()', function() {
    it('respond with matching records', function(done) {
      db.find({type: 'User'}, function(err, res) {
        if (err) return done(err);
        res.should.have.length(3);
        done();
      });
    });
  });
});
```

### ROOT-LEVEL HOOKS
你可以选择几个文件来添加根级别的钩子。例如，添加`beforeEach()`在所有`describe()`块外面(译者注：可以理解为最顶级作用域中)，这会造成在每个测试用例之前调用这个钩子函数。不仅仅它所在的这个文件(这是因为Mocha有一个暗藏的`describe()`，叫做"root-suite")。

```javascript
beforeEach(function () {
    console.log('before every test in every file');
})
```

### DELAYED ROOT SUITE
如果想在mocha命令运行之后，先做一些别的工作，再启动测试，可以使用mocha --delay命令，此命令会在全局环境中生成一个run函数，延迟工作完成后调用run函数即可启动测试。

```javascript
setTimeout(function () {
    // do some setup
    
    describe('my suite', function () {
        // ...
    });
    
    run();
}, 5000)
```
### PENDING TESTS
不给测试用例传递一个回调函数，就是被等待实现的测试用例，但同样会在报告中体现出来。

```javascript
describe('Array', function() {
    describe('#indexOf', function () {
        // 等待测试
        it('should return -1 when the value is nor present');
    });
});
```

### EXCLUSIVE TESTS
在用例测试集或者用例单元后面加上`.only()`方法，可以让mocha只测试此用例集合或者用例单元。下面是一个仅执行一个特殊的测试单元的例子：

```javascript
describe('Array', function () {
    describe.only('#indexOf()', function () {
        // ....
    })
})
```
注意：在Array用例集下面嵌套的集合，只有#indexOf用例集合会被执行。

下面的这个例子是仅仅执行唯一一个测试单元。

```javascript
describe('Array', function() {
    describe('#indexOf', function() {
        it.only('should return -1 unless preset', function () {
            // ...
        })
        it('should return the index when present', function () {
            // ...
        })
    })
})
```
在v3.0.0版本之前，`.only()`函数通过字符串匹配的方式去决定哪个测试应该被执行。但是在v3.0.0版本及以后，`.only()`可以被定义多次来定义一系列的测试子集。

```javascript
describe('Array', function () {
    describe('#indexOf', function () {
        it.only('should return -1 unless present', function () {
            // this test will be run
        })
        it.only('should return index when present', function () {
            // this test will also be run
        })
        it('should return -1 if called with a non-Array context', function () {
            // this test will not be run
        })
    })
})
```

你也可以选择多个测试集合：

```javascript

describe('Array', function () {
    describe.only('#indexOf()', function () {
        it('should return -1 unless present', function() {
          // this test will be run
        });
    
        it('should return the index when present', function() {
          // this test will also be run
        });
    });
    describe.only('#concat()', function () {
        it('should return a new Array', function () {
          // this test will also be run
        });
    });
    
    describe('#slice()', function () {
        it('should return a new Array', function () {
          // this test will not be run
        });
    });
})

```
上面两种情况也可以结合在一起使用：

```javascript
describe('Array', function () {
    describe.only('#indexOf()', function () {
        it.only('should return -1 unless present', function () {
            // this test will be run
        })
        it('should return the index when present', function () {
            // this test will not be run
        })
    })
})
```
注意：如果有钩子函数，钩子函数会被执行。

> 除非你是真的需要它，否则不要提交`only()`到你的版本控制中。

### INCLUSIVE TESTS
和`only()`方法相反，`.skip()`方法可以用于跳过某些测试测试集合和测试用例。所有被跳过的用例都会被标记为`pending`用例，在报告中也会以`pending`用例显示。下面是一个跳过整个测试集的例子。

```javascript
describe('Array', function () {
    describe.skip('#indexOf', function () {
        // ...
    })
})
```
或者指定跳过某一个测试用例：

```javascript
describe('Array', function () {
    describe('#indexOf()', function () {
        it.skip('should return -1 unless present', function () {
            // this test will not be run
        })
        
        it('should return the index when present', function () {
            // this test will be run
        })
    })
})
```

> 最佳实践：使用`.skip()`方法来跳过某些不需要的测试用例而不是从代码中注释掉。

有些时候，测试用例需要某些特定的环境或者一些特殊的配置，但我们事先是无法确定的。这个时候，我们可以使用`this.skip()`［译者注：这个时候就不能使用箭头函数了］根据条件在运行的时候跳过某些测试用例。

```javascript
it('should only test in the correct environment', function () {
    if(/* check the environment */) {
        // make assertions
    } else {
        this.skip()
    }
})
```
这个测试在报告中会以`pending`状态呈现。为了避免测试逻辑混乱，在调用skip函数之后，就不要再在用例函数或after钩子中执行更多的逻辑了。

下面的这个测试和上面的相比，因为没有在else分支做任何事情，当if条件不满足的时候，它仍然会在报告中显示passing。

```javascript
it('should only test in the correct environment', function () {
    if (/* check test environment */) {
        // make assertion
    } else {
        // do nothing
    }
})
```
 
 > 最佳事件：千万不要什么事情都不做，一个测试应该做个断言判断或者使用skip()

 我们也可以在before钩子函数中使用.skip()来跳过多个测试用例或者测试集合。

 ```javascript
before(function () {
    if(/* check test environment */) {
        // setup mode
    } else {
        this.skip()
    }
})
 ```

 > Mocha v3.0.0之前，在异步的测试用例和钩子函数中是不支持`this.skip()`的。

### RETRY TESTS
Mocha允许你为失败的测试用例指定需要重复的次数。这个功能是为端对端测试所设计的，因为这些测试的数据不好模拟。Mocha不推荐在单元测试中使用这个功能。

这个功能会重新运行beforeEach/afterEach钩子，但不会重新运行before/after钩子。

下面是一个使用Selenium webdriver写的一个重复执行的测试用例。

```javascript
describe('retries', function () {
    // 尝试全部的失败的测试4次，
    this.retries(4);

  beforeEach(function () {
    browser.get('http://www.yahoo.com');
  });

  it('should succeed on the 3rd try', function () {
    // Specify this test to only retry up to 2 times
    this.retries(2);
    expect($('.foo').isDisplayed()).to.eventually.be.true;
  });
})
```

### DYNAMICALLY GENERATING TESTS
Mocha可以使用`Function.prototype.call`和函数表达式来定义测试用例，其实就是动态生成一些测试用例，不需要使用什么特殊的语法。和你见过的其他框架可能有所不同，这个特性可以通过定义一些参数来实现测试用例所拥有的功能。

```javascript
var assert = require('chai').assert;

function add() {
  return Array.prototype.slice.call(arguments).reduce(function(prev, curr) {
    return prev + curr;
  }, 0);
}

describe('add()', function() {
  var tests = [
    {args: [1, 2],       expected: 3},
    {args: [1, 2, 3],    expected: 6},
    {args: [1, 2, 3, 4], expected: 10}
  ];

  tests.forEach(function(test) {
    it('correctly adds ' + test.args.length + ' args', function() {
      var res = add.apply(null, test.args);
      assert.equal(res, test.expected);
    });
  });
});
```

上面的测试用例所产生的结果如下：

```bash
$ mocha

  add()
    ✓ correctly adds 2 args
    ✓ correctly adds 3 args
    ✓ correctly adds 4 args
```

### TEST DURATION
很多的测试报告都会显示测试所花费的时间，同样也会对一些耗时的测试作出特殊的标记。

![](http://ww1.sinaimg.cn/large/006FmM8yly1fjrcs0bgo3j30i70bgglh.jpg)

我们可以使用slow()方法来明确的表示出，超过多久的时间，这个测试就可以认为是slow的。

```javascript
describe('something slow', function() {
  this.slow(10000);

  it('should take long enough for me to go make a sandwich', function() {
    // ...
  });
});
```

### TIMEOUTS

测试集合超时:

在测试集合上定义超时时间，会对这个测试集合中所有的测试用例和测试集合起作用。我们可以通过`this.timeout(0)`来关闭超时判断的功能。而且在测试用例和测试集合上定义的超时时间会覆盖外围的测试集合的设置。

```javascript
describe('a suite of tests', function() {
  this.timeout(500);

  it('should take less than 500ms', function(done){
    setTimeout(done, 300);
  });

  it('should take less than 500ms as well', function(done){
    setTimeout(done, 250);
  });
})
```

测试用例超时：

我们也可以给测试用例定义超时时间，或者通过`this.timeout(0)`来禁止超时时间的判断。

```python
it('should take less than 500ms', function(done){
  this.timeout(500);
  setTimeout(done, 300);
});
```

钩子函数超时：

也可以给钩子函数设定超时时间，同样也可以使用`this.timeout(0)`来禁止掉超时时间的判断。

```javascript
describe('a suite of tests', function() {
  beforeEach(function(done) {
    this.timeout(3000); // A very long environment setup.
    setTimeout(done, 2500);
  });
});
```

> 在Mocha v3.0.0版本及以上，如果设定的超时时间比[最大延迟时间](https://developer.mozilla.org/docs/Web/API/WindowTimers/setTimeout#Maximum_delay_value)的值大，那么也会被认为是禁止掉超时时间的判断。

### DIFFS

如果做断言的时候抛出了`AssertionErrors`的异常，且错误对象中含有`err.expected`属性和`err.actual`属性，mocha会在报告中展示出期望值和实际值之间的差异。

![](http://ww1.sinaimg.cn/large/006FmM8yly1fjtuotx6q6j30gg0b6767.jpg)

### USAGE

```bash
格式：mocha [debug] [options] [files]

命令：
    init <path> : 生成一个在浏览器中运行的单元测试的模版
```
当我们运行如下命令的时候:`mocha init .`会在当前路径中生成一个模版，文件如下：

![](http://ww1.sinaimg.cn/large/006FmM8yly1fjtuxg08gnj30hv03gq3h.jpg)

mocha的命令的基本选项：

```bash
Options:

    -h, --help                  输出帮助信息
    -V, --version               输出mocha的版本号
    -A, --async-only            强制所有的测试用例必须使用callback或者返回一个promise的格式来确定异步的正确性
    -c, --colors                在报告中显示颜色
    -C, --no-colors             在报告中禁止显示颜色
    -g, --growl                 在桌面上显示测试报告的结果
    -O, --reporter-options <k=v,k2=v2,...>  设置报告的基本选项
    -R, --reporter <name>       指定测试报告的格式
    -S, --sort                  对测试文件进行排序
    -b, --bail                  在第一个测试没有通过的时候就停止执行后面所有的测试
    -d, --debug                 启用node的debugger功能
    -g, --grep <pattern>        用于搜索测试用例的名称，然后只执行匹配的测试用例
    -f, --fgrep <string>        只执行测试用例的名称中含有string的测试用例
    -gc, --expose-gc            展示垃圾回收的log内容
    -i, --invert                只运行不符合条件的测试用例，必须和--grep或--fgrep之一同时运行
    -r, --require <name>        require指定模块
    -s, --slow <ms>             指定slow的时间，单位是ms，默认是75ms
    -t, --timeout <ms>          指定超时时间，单位是ms，默认是200ms
    -u, --ui <name>             指定user-interface (bdd|tdd|exports)中的一种
    -w, --watch                 用来监视指定的测试脚本。只要测试脚本有变化，就会自动运行Mocha
    --check-leaks               检测全局变量造成的内存泄漏问题
    --full-trace                展示完整的错误栈信息
    --compilers <ext>:<module>,...  使用给定的模块来编译文件
    --debug-brk                 启用nodejs的debug模式
    --es_staging                启用全部staged特性
    --harmony<_classes,_generators,...>     all node --harmony* flags are available
    --preserve-symlinks                     告知模块加载器在解析和缓存模块的时候，保留模块本身的软链接信息
    --icu-data-dir                          include ICU data
    --inline-diffs              用内联的方式展示actual/expected之间的不同
    --inspect                   激活chrome浏览器的控制台
    --interfaces                展示所有可用的接口
    --no-deprecation            不展示warning信息
    --no-exit                   require a clean shutdown of the event loop: mocha will not call process.exit
    --no-timeouts               禁用超时功能
    --opts <path>               定义option文件路径 
    --perf-basic-prof           启用linux的分析功能
    --prof                      打印出统计分析信息
    --recursive                 包含子目录中的测试用例
    --reporters                 展示所有可以使用的测试报告的名称
    --retries <times>           设置对于失败的测试用例的尝试的次数
    --throw-deprecation         无论任何时候使用过时的函数都抛出一个异常
    --trace                     追踪函数的调用过程
    --trace-deprecation         展示追踪错误栈
    --use_strict                强制使用严格模式
    --watch-extensions <ext>,... --watch监控的扩展 
    --delay                     异步测试用例的延迟时间
```

#### About Babel
如果你在js文件中使用了es6的模块，你可以`npm install --save-dev babel-register`，然后使用--require选项` mocha --require babel-register`。如果你指定了文件的后缀名，--compilers选项也是必需的。

-b, \-\-bail
--
如果你只对第一个异常感兴趣，可以使用这个选项。

-d, \-\-debug
--
启用nodejs的debug功能。这个选项会用`node debug <file>`的模式运行你的脚本，所以会在`debugger`语句处暂停执行。这个选项和mocha debug以及mocha --debug是不同的；mocha debug将会唤起nodejs默认的debug客户端，mocha --debug也可以使用不同的接口，比如－－Blink的控制台工具。

\-\-globals names
--
names是一个逗号分隔的列表，例如，假设你的app需要使用全局变量`app`和`YUI`，这个时候你就可以使用`--global app, YUI`了。names也可以是一个通配符。比如，--global '\*bar'将会匹配foobar，barbar等。参数传入 * 的话，会忽略所有全局变量。

\-\-check-leaks
--
默认情况下，mocha并不会去检查应用暴露出来的全局变量，加上这个配置后就会去检查，此时某全局变量如果没有用上面的--GLOBALS去配置为可接受，mocha就会报错。

-r, \-\-require module-name
--
这个命令可以引入一些测试运行时候所必需的依赖。比如should.js，通过这个选项你不需要在每个文件使用require('should')来添加should.js了。也可以用--require ./test/helper.js这样的命令去引入指定的本地模块。
但是，如果要引用模块导出的对象，还是需要require，var should = require('should')这样搞。

-u, \-\-ui name
--
用来指定测试所使用的接口，默认是'bdd'。

-R, \-\-reporter name
--
这个命令用于指定报告的格式。默认是spec。这个选项也可以用于指定使用第三方的报告样式。例如，在`npm install mocha-lcov-reporter`后，就可以使用`--reporter mocha-lcov-reporter`来指定报告格式。

-t, \-\-timeout ms
--
用来指定用例超时时间。单位是ms，默认是2s。可以直接使用带单位的时间来覆盖掉默认的单位。例如：--timeout 2s和--timeout 2000是一样的。

-s, \-\-slow ms
--
用来指定慢用例判定时间，默认是75ms。

-g, \-\-grep
--
参数用于搜索测试用例的名称（即it块的第一个参数），然后只执行匹配的测试用例。

```javascript
describe('api', function() {
  describe('GET /api/users', function() {
    it('respond with an array of users', function() {
      // ...
    });
  });
});

describe('app', function() {
  describe('GET /users', function() {
    it('respond with an array of users', function() {
      // ...
    });
  });
});
```
当我们使用`--grep api`或者`--grep app`只能运行其中一个对应的测试。
