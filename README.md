# bpaf

Parse command line arguments by composing a parser from the components

# Usage

Add `bpaf` under `[dependencies]` in your `Cargo.toml`

```toml
[dependencies]
bpaf = "0.1"
```

Define fields used in parser

```rust
let speed_in_kph
    = short('k').long("speed_kph")         // give it a name
      .argument()                          // it's an argument
      .metavar("SPEED")                    // with metavar
      .help(("speed in KPH").build()       // and help message
      .parse(|s| f64::from_str(&s).map(|s| s / 0.62)); // that is parsed from string

let speed_in_mph
    = short('m').long("speed_mph")
      .argument()
      .metavar("SPEED")
      .help(("speed in KPH").build()
      .parse(|s| f64::from_str(&s));
```

Arguments can be composed in multiple ways, for example
if application wants speed - it can accept it in either of two formats

```rust
let speed: Parser<f64> = speed_in_kph.or_else(speed_in_mph);
```

As far as the rest of the application is concerned - there's only one parameter

# main features


## full help is generated including usage line and a list of commands
```
Usage: [-a|--AAAAA] (-b) (-m|--mph) | (-k|--kph) COMMAND

this is a test

Available options:
    -a, AAAAA        maps to a boolean, is optional
    -b               also maps to a boolean but mandatory
    -m, mph <SPEED>  speed in MPH
    -k, kph <SPEED>  speed in KPH
    -h, help         Prints help information
    -v, version      Prints version information

Available commands:
    accel  command for acceleration
```

## alternatives and parse failures are handled at parsing level:

```rust

let kph = short('k').argument().metavar("SPEED").help("speed in KPH").build()
        .parse(|s| f64::from_string(s)?).guard(|s| s > 0);
let mph = short('m').argument().metavar("SPEED").help("speed in MPH").build()
        .parse(|s| f64::from_string(s)?  * 1.6).guard(|s| s > 0);
let speed = kph.or(mph);
```

## composable and reusable
```
fn get_something() -> Parser<Foo> {
   // define flag/argument/positional here with help and everything related
   // it's a usual function which you can export and reuse across multiple
   // applications
}
```


# fast compilation time

library provides a small number of components that can be composed
in a multiple ways on user side


# compare vs

## vs clap

gives parsed data back instead of stringly typed value

```rust
    ...
      .arg(Arg::with_name("v")
           .short("v")
           .multiple(true)
           .help("Sets the level of verbosity"))


    // Vary the output based on how many times the user used the "verbose" flag
    // (i.e. 'myprog -v -v -v' or 'myprog -vvv' vs 'myprog -v'
    match matches.occurrences_of("v") {
        0 => println!("No verbose info"),
        1 => println!("Some verbose info"),
        2 => println!("Tons of verbose info"),
        3 | _ => println!("Don't be crazy"),
    }

```

```rust

    short('v').req_flag()
        .help("Sets the level of verbosity")
        .many().parse(|xs|xs.len())
        .guard(|x| x <= 3)
    ...

    match v {
        0 => println!("No verbose info"),
        1 => println!("Some verbose info"),
        2 => println!("Tons of verbose info"),
        _ => unreachable!(),
    }
```

## vs pico-args

- generates help message
- handles alternatives and subcommands


## vs macro based one

vs `structopt` `gumdrop` `argh`

- no proc macros:
- no syn/quote/proc-macro2 dependencies => faster compilation time
- pure rust, no cryptic commands, full support from tools