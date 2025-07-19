# Grammar File

Felys grammar file uses customized [syntax](../kickstart/syntax.md) to generate the parser and lexer. However, due to limitations of the parser generator, the non-terminals `ident` and `eof` are still written by hand. Their implementations can be found in the source code.

```
{
    use crate::ast::*;
    #[allow(unused)]
    use std::rc::Rc;
}

peg grammar -> { Grammar }:
    / stmts=(stmt=stmt $)* #eof { Grammar(stmts) }
    ;

@(memo)
peg pat -> { Pat }:
    / UNDERSCORE { Pat::Any }
    / ident=ident { Pat::Ident(ident) }
    / lit=lit { Pat::Lit(lit) }
    / '(' first=pat ',' second=pat more=(',' pat=pat)* ','? ')' {
        Pat::Tuple(BufVec::new([first, second], more))
    }
    ;

peg stmt -> { Stmt }:
    / expr=expr ';' { Stmt::Semi(expr) }
    / expr=expr { Stmt::Expr(expr) }
    / ';' { Stmt::Empty }
    ;

peg block -> { Block }:
    / '{' stmts=stmt* #'}' { Block(stmts) }
    ;

@(memo)
peg expr -> { Expr }:
    / assignment=assignment
    / disjunction=disjunction
    / block=block { Expr::Block(block) }
    / BREAK expr=expr? { Expr::Break(expr.map(Rc::new)) }
    / CONTINUE { Expr::Continue }
    / FOR pat=#pat #IN expr=#expr block=#block { Expr::For(pat, expr.into(), block) }
    / IF expr=#expr block=#block otherwise=(ELSE expr=#expr)? {
        Expr::If(expr.into(), block, otherwise.map(Rc::new))
    }
    / LOOP block=#block { Expr::Loop(block) }
    / RETURN expr=expr? { Expr::Return(expr.map(Rc::new)) }
    / WHILE expr=#expr block=#block { Expr::While(expr.into(), block) }
    / STEP loss=#expr #BY lr=#expr { Expr::Step(loss.into(), lr.into()) }
    / PRINT expr=#expr { Expr::Print(expr.into()) }
    ;

peg assignment -> { Expr }:
    / pat=pat '=' !'=' expr=#expr { Expr::Assign(pat, AssOp::Eq, expr.into()) }
    / pat=pat '+=' expr=#expr { Expr::Assign(pat, AssOp::AddEq, expr.into()) }
    / pat=pat '-=' expr=#expr { Expr::Assign(pat, AssOp::SubEq, expr.into()) }
    / pat=pat '*=' expr=#expr { Expr::Assign(pat, AssOp::MulEq, expr.into()) }
    / pat=pat '/=' expr=#expr { Expr::Assign(pat, AssOp::DivEq, expr.into()) }
    / pat=pat '%=' expr=#expr { Expr::Assign(pat, AssOp::ModEq, expr.into()) }
    ;

peg disjunction -> { Expr }:
    / lhs=disjunction OR rhs=#conjunction {
        Expr::Binary(lhs.into(), BinOp::Or, rhs.into())
    }
    / conjunction=conjunction
    ;

peg conjunction -> { Expr }:
    / lhs=conjunction AND rhs=#inversion {
        Expr::Binary(lhs.into(), BinOp::And, rhs.into())
    }
    / inversion=inversion
    ;

peg inversion -> { Expr }:
    / NOT inversion=#inversion { Expr::Unary(UnaOp::Not, inversion.into()) }
    / equality=equality
    ;

peg equality -> { Expr }:
    / lhs=equality '==' rhs=#comparison { Expr::Binary(lhs.into(), BinOp::Eq, rhs.into()) }
    / lhs=equality '!=' rhs=#comparison { Expr::Binary(lhs.into(), BinOp::Ne, rhs.into()) }
    / comparison=comparison
    ;

peg comparison -> { Expr }:
    / lhs=comparison '>=' rhs=#term { Expr::Binary(lhs.into(), BinOp::Ge, rhs.into()) }
    / lhs=comparison '<=' rhs=#term { Expr::Binary(lhs.into(), BinOp::Le, rhs.into()) }
    / lhs=comparison '>' rhs=#term { Expr::Binary(lhs.into(), BinOp::Gt, rhs.into()) }
    / lhs=comparison '<' rhs=#term { Expr::Binary(lhs.into(), BinOp::Lt, rhs.into()) }
    / term=term
    ;

peg term -> { Expr }:
    / lhs=term '+' rhs=#factor { Expr::Binary(lhs.into(), BinOp::Add, rhs.into()) }
    / lhs=term '-' rhs=#factor { Expr::Binary(lhs.into(), BinOp::Sub, rhs.into()) }
    / factor=factor
    ;

peg factor -> { Expr }:
    / lhs=factor '*' rhs=#dot { Expr::Binary(lhs.into(), BinOp::Mul, rhs.into()) }
    / lhs=factor '/' rhs=#dot { Expr::Binary(lhs.into(), BinOp::Div, rhs.into()) }
    / lhs=factor '%' rhs=#dot { Expr::Binary(lhs.into(), BinOp::Mod, rhs.into()) }
    / dot=dot
    ;

peg dot -> { Expr }:
    / lhs=dot '@' rhs=#unary { Expr::Binary(lhs.into(), BinOp::Dot, rhs.into()) }
    / unary=unary
    ;

peg unary -> { Expr }:
    / '+' unary=#unary { Expr::Unary(UnaOp::Pos, unary.into()) }
    / '-' unary=#unary { Expr::Unary(UnaOp::Neg, unary.into()) }
    / call=call
    ;

peg call -> { Expr }:
    / call=call '(' args=args? #')' {
        Expr::Call(call.into(), args)
    }
    / primary=primary
    ;

peg args -> { BufVec<Expr, 1> }:
    / first=expr more=(',' expr=expr)* ','? { BufVec::new([first], more) }
    ;

peg primary -> { Expr }:
    / lit=lit { Expr::Lit(lit) }
    / ident=ident { Expr::Ident(ident) }
    / RUST ident=#ident { Expr::Rust(ident) }
    / '(' expr=#expr ')' { Expr::Paren(expr.into()) }
    / '(' first=#expr #',' second=#expr more=(',' expr=expr)* ','? #')' {
        Expr::Tuple(BufVec::new([first, second], more))
    }
    / '[' first=row more=row* #']' { Expr::Matrix(BufVec::new([first], more)) }
    / '[' args=args? #']' { Expr::List(args) }
    / '|' params=params? #'|' expr=#expr { Expr::Closure(params, expr.into()) }
    / '<' rows=#INT #',' cols=#INT #'>' { Expr::Param(rows, cols, x.stream.cursor) }
    ;

peg row -> { BufVec<Float, 1> }:
    / first=FLOAT more=(',' f=FLOAT)* ';' { BufVec::new([first], more) }
    ;

peg params -> { BufVec<Ident, 1> }:
    / first=ident more=(',' ident=ident)* ','? { BufVec::new([first], more) }
    ;

peg lit -> { Lit }:
    / float=FLOAT { Lit::Float(float) }
    / int=INT { Lit::Int(int) }
    / str=STR { Lit::Str(str) }
    / bool=BOOL { Lit::Bool(bool) }
    ;

lex FLOAT -> { Float }: float=F64 { Float(float) } ;

lex INT -> { Int }: int=USIZE { Int(int) } ;

lex STR -> { Vec<Chunk> }: '"' chunks=CHUNK* #'"' ;

lex CHUNK -> { Chunk }:
    / slice=SLICE { Chunk::Slice(slice) }
    / '\\u{' hex=#HEX #'}' { Chunk::Unicode(hex) }
    / '\\' escape=#ESCAPE { Chunk::Escape(escape) }
    ;

lex BOOL -> { Bool }:
    / "true" !TAIL { Bool::True }
    / "false" !TAIL { Bool::False }
    ;

lex UNDERSCORE -> { () }: "_" !TAIL ;
lex BREAK -> { () }: "break" !TAIL ;
lex CONTINUE -> { () }: "continue" !TAIL ;
lex FOR -> { () }: "for" !TAIL ;
lex IN -> { () }: "in" !TAIL ;
lex IF -> { () }: "if" !TAIL ;
lex LOOP -> { () }: "loop" !TAIL ;
lex RETURN -> { () }: "return" !TAIL ;
lex WHILE -> { () }: "while" !TAIL ;
lex ELSE -> { () }: "else" !TAIL ;
lex OR -> { () }: "or" !TAIL ;
lex AND -> { () }: "and" !TAIL ;
lex NOT -> { () }: "not" !TAIL ;
lex STEP -> { () }: "step" !TAIL ;
lex BY -> { () }: "by" !TAIL ;
lex PRINT -> { () }: "print" !TAIL ;
lex RUST -> { () }: "rust" !TAIL ;

@(intern, memo)
IDENT: [a-zA-Z_] TAIL* ;
TAIL: [a-zA-Z0-9_] ;

@(intern) {
    USIZE: '0' | [1-9][0-9]* ;
    F64: [1-9][0-9]* '.' [0-9]+ | '0.' [0-9]+;
    SLICE: [^"\\]+ ;
    HEX: [0-9a-f]* ;
    ESCAPE: ['ntr\\] ;
}

@(ws) {
    WS: [\u{20}\n\t\r]+ ;
    COMMENT: '//' [^\n]* ;
}
```
