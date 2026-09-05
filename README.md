# dry4py

> 「4py 系列」:[crap4py](http://lzxsvn:3000/qinyuanj/crap4py) · [dry4py](http://lzxsvn:3000/qinyuanj/dry4py) · [mutate4py](http://lzxsvn:3000/qinyuanj/mutate4py)
> 对应「4lua 系列」:[dry4lua](http://lzxsvn:3000/eggy/dry4lua)

`dry4py` 是 Structural duplicate detector for Python (AST normalization + Jaccard)。它是「4lua 系列」的 Python 同构实现:
上游 unclebob 的 dry4clj/go/java 规格以 Python 忠实实现(「零本地独有发明」教义;
Python 无上游规格的部分以本文档为规格,对齐不变量逐条列出,偏离注明理由)。

库自包含:`src/dry4py/` 下不硬编码任何宿主专属路径;共享基础件
(common / covdata / astnorm)vendor 在 `_internal/`,不是公共 API。
源自 [ugc-image-tool](http://lzxsvn:3000/qinyuanj/ugc-image-tool) 的 `tools/packages/dry`。

## 安装与入口

```sh
pip install git+http://lzxsvn:3000/qinyuanj/dry4py.git
python -m dry4py --help        # 或直接 dry4py --help
```

在宿主项目根目录运行;宿主解释器优先取 `.venv/bin/python`,否则当前解释器。


## dry(对齐不变量)

- 默认参数 `threshold 0.82 / min-lines 4 / min-nodes 20` 及选项集。
- 指纹 = 序列化后的归一化子树集合;相似度 = Jaccard。
- 检测单位 = 函数作用域(模块级函数 + 类方法)。
- 文本输出骨架(`DUPLICATE score=%.2f` + 两行 `file:start-end`,无候选时输出
  `No duplicate candidates found.`)以及未知格式时的退出码 2。
- 归一化将运算符/关键字保留在标签中,同时剥离标识符和字面量(与 dry4go 最接近)。
- 嵌套函数在父级指纹中折叠为 `(function)` 叶节点;同一文件行号重叠条目排除
  (对齐 dry4java `overlaps` 语义)。

```sh
python tools/cli.py dry [--threshold N] [--min-lines N] [--min-nodes N]
                        [--format text|json] [--limit N] [file-or-directory ...]
```

**有意偏离**:JSON 输出带函数 `name` 字段;文本输出提供 `--limit`(均同 lua 版)。

## 布局

```text
src/dry4py/
  cli.py          # 选项走查与退出码
  engine.py       # 核心算法
  _internal/      # vendor 的共享基础件:common / astnorm
tests/            # pytest;test_internal_* 覆盖 _internal
```

## 测试

```sh
python3 -m venv .venv && .venv/bin/pip install -e '.[test]'
.venv/bin/python -m pytest
```
