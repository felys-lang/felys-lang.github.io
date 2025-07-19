# Meta Syntax

This is a fully functional meta-syntax that can bootstrap the parser generator. The non-terminals `action` and `eof` are handwritten. The former requires embedding a foreign language, so it needs to maintain a brace counter, which is non-standard.

```
{
    use crate::ast::*;
    #[allow(unused_imports)]
    use crate::builder::common::s2c;
}

peg grammar -> { Grammar }:
    / import=action? $ callables=(callable=callable $)* #eof {
        Grammar { import, callables }
    }
    ;

peg callable -> { Callable }:
    / decorator=decorator? prefix=PREFIX name=#NAME #'->' ty=#action #':' rule=#rule #';' {
        Callable::Rule(decorator, prefix, name, ty, rule)
    }
    / decorator=decorator? name=NAME #':' regex=#union #';' {
        Callable::Regex(decorator, name, regex)
    }
    / decorator=decorator #'{' shared=#(name=NAME #':' regex=#union #';')+ #'}' {
        Callable::Shared(decorator, shared)
    }
    ;

peg decorator -> { Decorator }:
    / '@' #'(' first=#tag more=(',' tag=#tag)* #')' { Decorator { first, more } }
    ;

peg tag -> { Tag }:
    / 'memo' { Tag::Memo }
    / 'left' { Tag::Left }
    / 'intern' { Tag::Intern }
    / 'ws' { Tag::Whitespace }
    ;

peg rule -> { Rule }:
    / '/'? first=#alter more=('/' alter=#alter)* {
        Rule { first, more }
    }
    ;

peg alter -> { Alter }:
    / assignments=#assignment+ action=action? { Alter { assignments, action } }
    ;

peg assignment -> { Assignment }:
    / name=NAME '=' item=#item { Assignment::Named(name, item) }
    / lookahead=lookahead { Assignment::Lookahead(lookahead) }
    / item=item { Assignment::Anonymous(item) }
    / '$' { Assignment::Clean }
    ;

peg lookahead -> { Lookahead }:
    / '&' atom=#atom { Lookahead::Positive(atom) }
    / '!' atom=#atom { Lookahead::Negative(atom) }
    ;

peg item -> { Item }:
    / atom=atom '?' { Item::Optional(atom) }
    / atom=atom '*' { Item::ZeroOrMore(atom) }
    / eager='#'? atom=atom '+' { Item::OnceOrMore(eager.is_some(), atom) }
    / eager='#'? atom=atom { Item::Name(eager.is_some(), atom) }
    ;

peg atom -> { Atom }:
    / '(' rule=#rule #')' { Atom::Nested(rule) }
    / expect=EXPECT { Atom::Expect(expect) }
    / name=NAME { Atom::Name(name) }
    ;

peg union -> { Regex }:
    / lhs=union '|' rhs=#concat { Regex::Union(lhs.into(), rhs.into()) }
    / concat=concat { concat }
    ;

peg concat -> { Regex }:
    / lhs=concat rhs=repeat { Regex::Concat(lhs.into(), rhs.into()) }
    / repeat=repeat { repeat }
    ;

peg repeat -> { Regex }:
    / inner=repeat '+' { Regex::OnceOrMore(inner.into()) }
    / inner=repeat '*' { Regex::ZeroOrMore(inner.into()) }
    / primary=primary { Regex::Primary(primary) }
    ;

peg primary -> { Primary }:
    / '(' regex=#union #')' { Primary::Parentheses(regex.into()) }
    / string=STRING { Primary::Literal(string) }
    / name=NAME { Primary::Name(name) }
    / set=SET { set }
    ;

lex PREFIX -> { Prefix }:
    / 'peg' !TAIL { Prefix::Peg }
    / 'lex' !TAIL { Prefix::Lex }
    ;

lex SET -> { Primary }:
    / '[' '^' set=#RANGE+ #']' { Primary::Exclude(set) }
    / '[' set=#RANGE+ #']' { Primary::Include(set) }
    ;

lex EXPECT -> { Expect }:
    / '\'' expect=#SQC+ #'\'' { {
        let expect = expect
            .iter()
            .map(|c| s2c(x.intern.get(c).unwrap()))
            .collect::<String>();
        let id = x.intern.id(expect.as_str());
        Expect::Once(id)
    } }
    / '"' expect=#DQC+ #'"' { {
        let expect = expect
            .iter()
            .map(|c| s2c(x.intern.get(c).unwrap()))
            .collect::<String>();
        let id = x.intern.id(expect.as_str());
        Expect::Keyword(id)
    } }
    ;

lex STRING -> { Vec<usize> }:
    / '\'' string=#SQC+ #'\'' { string }
    ;

lex RANGE -> { (usize, usize) }:
    / start=BC '-' end=#BC { (start, end) }
    / ch=BC { (ch, ch) }
    ;

@(intern, memo)
NAME: [a-zA-Z_] TAIL* ;
TAIL: [a-zA-Z0-9_] ;

@(intern) {
    DQC: UNICODE | ESCAPE | [^\"\\] ;
    SQC: UNICODE | ESCAPE | [^\'\\] ;
    BC: UNICODE | ESCAPE | [^\]\\] ;
}

@(ws) {
    WS: [\u{20}\n\t\r]+ ;
    COMMENT: '//' [^\n]* ;
}

UNICODE: '\\' 'u' '{' [0-9a-f]* '}' ;
ESCAPE: '\\' [\'\"\[\]ntr\\] ;
```
