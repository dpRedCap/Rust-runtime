// Week 5

// Section #1 Error handling

//Intro

// Ok, at this moment, Rust should be start irritating you with it's attention to details and complaining compiler.
// But I hope after today lecture, everything starts to make sense.

// In general error-handling is the topic mostly omitted in programming classes, with at most treating errors with some
// print messages or exceptions, well at the time of first encountering programming concepts, probably it's ok.

// Know it's time to turn tables and understand what's all this about Error Handling.

// Rust Error-handling system is pretty nice. And since Rust is a modern language, it adopted nice practices, by the way this error handling system is not new (Golang,Swift, Kotlin also have a similar system).

// Before we start, let me share you a general phylosophy of this error handling systems, so that examples  from today, will fit into the right place into your mind.

// And this philosophy is similar as some people handle problems => "Delay the solving problem as long as possible or return problem to higher level"

// Let me give you an example. May be it's not the best, but it helps me to create a mind model in my brain.

// Imagine a situation in a restaurant.

// Customers(MMA fighters) refuses to pay the bill!

// Waiter(needs to deal with issue) don't punch the customer and don't yell at them, the best the waiter can do is ask for manager help

// Manager(needs to deal with issue), let's say the manager, don't know what to do he calls the police

// Police Patrol(needs to deal with issue), let's say patrol can't do anything and situation escalates further

// Squad came (last instance, deals with the issue)

// This situation is pretty unrealistic but it resembles the error handling phylosophy.
// You need to try to solve the error by yourself or just pass the problem to higher instance
// Another important idea is to understand in that imaginary example, waiter could call the police immediately, but the question is
//"was it really necessary"


// Ok after this imaginary case, I believe you will start to immediately feel the philosphy of Rust for handling errors.

#[allow(unused_variables)]

// Rust models assumes two type of errors

// -> recoverable (Result<T,E>)
// -> unrecoverable error(panic!)

// So when code execution runs into problem it can panick(immediately stop execution) or (try to handle the error somehow, by returning Result enum )

fn main() {
    // Part 1 Reaction to Errors with panic!

    // =>panic macros our own 
    // println!("Everything is good");

    // panic!("Crash the program, stop running, clean the memory");

    // println!("Everything is good");

    // panic macros from Rust compiler

    //let v = vec![1,2,3];
    //v[4]; // Rust 100 procent sure that trying to access memory address, which does not belong to vector is a error it can not tolerate, so it panicks immediately

    // RUST_BACKTRACE=1 cargo run

    // Well that's pretty much it panicking and stoping execution immediately it's ok but not the solution, we need to try to resolve the issue till the last attempt before panicking

    //Part2 Deal with Errors!

    // Ok let's go with the long example, but really good from learning perspective

    // Problem: "Open file" -> straightforward

    use std::fs::File;

    let f = File::open("exam.txt");
    println!("{:?}",f); // well file exam.txt doesn't exist, but it still useful to run, and see output

    // We get back as a result for f variable error, but what is important to notice, no panick, our program executed till the end.

    // Techincally what it means, open method, is designed in way that, if there is no file to open it will not panick, it will just say well, there is some error, and whoever called me should now deal with this issue :) pretty fun

    // Let's intentionally get the the error, and see the data type being returned

    //let f: () = File::open("exam.txt");

    // Ok error messae shows that open function returns to us Result enum with (file,error)

    // so Result enum has two variants
    
/*
    enum Result<T, E> {
        Ok(T),
        Err(E),
    }

    Prety descriptive enum, if everything goes well make it OK, some problems occur treat it as Err

    // Ok at this point things start to become interesting, we know that match is a best way to handle all exaustive options
*/

    // let f = File::open("exam.txt");
    // let f = match f {
    //     Ok(file) => file,
    //     Err(error) => {
    //         panic!("Problem opening the file {:?}", error)
    //     },
    // };

    // Ok we were able to panick, than just continue running as in previous example, but still
    // something is missing, if file does'n exist may be we should create it

    // => Matching depends on error type

    //use std::fs::File;
    use std::io::ErrorKind;
    
    let f = File::open("exam.txt"); // Result of opening the file (in our case doesn't exist)

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("exam.txt") {// create also return Result which we need to handle
                
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}",other_error),
        },
    };

    // )) So far so good, we were able to recover from error

    // Ok at this point you may be thinking "this is insane":) 
    // Well I agree with you, in general if you write many lines of code to solve some simple problem, most likely there is a shortcut which you need to look for, but as I said, it much better to go through all nitty-gritty details at least once, so that later you will understand, what exactly is happening in other people codes.

     // => Shortcuts for Panic on Error: unwrap and expect

    // So instead all of this boilerplate code, we can use code which is implemented to Result<T, E> enum type -> unwrap()

    // https://github.com/rust-lang/rust/blob/53498eca50e25d8a11f9dc5859770715fa906fa7/src/libcore/result.rs#L684-L689

    // It's still the same matcher just implemented in Rust code

    // If everything is OK(result) => it will return result back, if Err() -> it will panick(crash your program)

    // let f = File::open("hello.txt").unwrap(); // this will fail or panick
    // or 
    
    // let f = File::open("hello.txt").expect("Failed to open hello.txt"); // stil fails, but allows to specify the error in text message 

    // If we open the source code, we will notice they both coded next to each other.

    // Ok after we have seen all of this examples for handling errors, let's come back to the philosphy of posponing reaction and just try to seek solutions from higher authorities.

    // This concept has official name : Propogating Errors.

    // Example: Imagine you have a (big brother)function which calls another (small brother)function and small function were not able to succeed and there is some error occured, well for him to panick is not good idea, so e just returns error to big brother and he should decide what to do next.

    // TextBook Example:

    // Problem try to read a user name from a File 

    // => First Version, just pack everything into one function:

    use std::io;
    use std::io::Read;
    use std::fs::File;

    fn read_username_from_file() -> Result<String, io::Error> { 
        // whoever calls us can be safe we will not crash the program, we just return the Error to caller
        let f = File::open("username.txt");
        let mut f = match f {
            Ok(file) => file,
            Err(e) => return Err(e), // dealing with error is not our responsobility we just return it
        };

        let mut s = String::new();

        match f.read_to_string(&mut s) { // read_to_string will read text into the passed string(or buffer), and return number of bytes it read
            Ok(_) => Ok(s), // we return passed string, placeholder is a result of number of bytes read, we don't need them
            Err(e) => Err(e),
        } // please notice, semicolon is omiited, because it's the last expression which will be automatically returned

    }

    
    // => Second version: using the ? Operator (Rust Shortcut) this works only for functions, that return enum Result

        fn read_username_from_file_2ver() -> Result<String, io::Error> {
            let mut f = File::open("hello.txt")?; // if fail will return error
            let mut s = String::new();
            f.read_to_string(&mut s)?; // if fail will return error
            Ok(s) // last expression need to wrap into Result enum Ok variant

            
        }
    
    // Third version: or chain methods, Well in general I believe you can write the code the way you want, 
    // but it's usually a very good skill, to be able to read someone's code.
    
        fn read_username_from_file_3ver() -> Result<String, io::Error> {
            let mut s = String::new();
            File::open("hello.txt")?.read_to_string(&mut s)?;
            Ok(s)
        }

    // Fourth version: "for real" recommended even by Rust documentation
    // https://doc.rust-lang.org/std/fs/fn.read_to_string.html
    
        use std::fs;
        fn read_username_from_file_4ver() -> Result<String, io::Error> {
            fs::read_to_string("hello.txt")
        }

        
    // And if you open source code, you will see that's it almost exactly as our 3rd version. (Like real implementation)

    // Well, what I can say, time spent investing in  understanding how something is work, will save you from many hours of meanigless Google search and debugging.

    
#[allow(unused_variables)]
#[allow(dead_code)]

// Section 2 : Beginining of SERIOUS PROGRAMMING

// Generic Types

// =>  Generics

// As you may noticed strongly type languages like Java, C++ is  a bit verbose on specifying data types. So it could happen, that there wll be some universal(general) approach how to processe some data, but that data can come in different data types.


// Just a remainder, don't take lightly some toy examples, and be focused

// as you may already noticed in programming, some simple ideas sometimes expand to the  unnatainable abstract complexity (if you solved some binary manipulation problems, you now how it works)


// Let's take simple scenario
// you have a function which iterates through vector and find the largest element



fn main() {

    // For integers 32, 
    fn largest_int(list: &[i32]) -> i32 {
        let mut largest = list[0];
        for &item in list.iter() {
            if item > largest {
                largest = item;
            }
        }
        largest
    }
    //Most straighforward function, so far so good.
    // But. it will not work for f32 or char.

    // Well, first solution which cames to mind of course is copy and paste

    // For floats 32, 
    fn largest_float(list: &[f32]) -> f32 {
        let mut largest = list[0];
        for &item in list.iter() {
            if item > largest {
                largest = item;
            }
        }

        largest
    }

    // For char, 
    fn largest_char(list: &[char]) -> char {
        let mut largest = list[0];
        for &item in list.iter() {
            if item > largest {
                largest = item;
            }
        }
        largest
    }

    // Well, usually you can smell, something is absolutely wrong, if I need to copy and paste almost the same code with just minor changes.

    // And you may think it will be so nice if there will be some generic type I can just put everywhere.

    // As you may already guessed, programmers are lazy and they indeed come up with some concepts to overcome this mindless copy pasting. 
    // But you need to keep in mind. There is absolutely no magic, and main logic doesn't dissappear, all the previous code, still should be copy pasted, but the main question who is gonna do it. You as a programmer or you gonna use programming language compiler to accomplish it for you in automatic manner.

    // Just remember, no magic, everything has a logic, but of course sometimes it can be beautiful))

    // so idea to provide some general data type is called Generics (I am pretty sure you can find even a better definition,but as a model, you can think of that like)

    // Basically, you are making announcment that you gonna treat any spherical type objects like (earth, ball, sun etc.) as a "Circle". I know it's a silly example, but this is describe the process pretty well.

    // => * Generic in functions:

    // First rewrite our function, and discuss why it may not work as expected

    // fn largest<T>(list: &[T]) -> T { 
    //     // after name of code entity in our case function, in <your name>, you are announcing, your universal data type
    //     // and use it everwhere, where you would normally specify concrete data type
        
    //     let mut largest = list[0];
    //     for &item in list.iter() {
    //         if item > largest {
    //             largest = item;
    //         }
    //     }
    //     largest
    // }

    // // T is just a convention for lazy programmers, as a type
    // // you can use whetever name you like

    // fn largestCircle<Circle>(list: &[Circle]) -> Circle { 
    //     let mut largest = list[0];
    //     for &item in list.iter() {
    //         if item > largest {
    //             largest = item;
    //         }
    //     }
    //     largest
    // }

    // Unfortunately our code as above will not compile, because it has a logic flow, because when we say T, we need to make 100% sure, that our code is generic enough to cover all possible scenarious, which we cannot guarantee yet.

    // For example, what if instead of &[int32] our slice &[Circle] will consist of Circle struct??? 
    // How in the world I suppose to compare them.

    // If you have programming long enough, you could say Oh I know I need to implement a comparator
    // Techincally right direction, but not yet.

    // Let's  come back to this idea a little bit later, but if you are curious I can give a hint (polymorphism and interfaces)


    // => * Generics in Struct Definitions

    
    #[derive(Debug)]
    
    struct Point<T> { // in <T> I am declaring that I am gonna use type T in my newly defined struct Point, compiler please compile for me all necessary boilerplate code, for whatever data type I will happen to be use in my programm development.

        
        x: T, // Notice, if I use T type for both fields, both of them will be the same type
        y: T,
    }

    let integer = Point{x:5,y:10};
    let float = Point{x:1.0,y:4.0};

    println!("int Point: {:?} float Point: {:?}",integer, float);


    // And of course there is no limitation for announcing generic types, but more that 2, it's weird : )
    
    #[derive(Debug)]
    struct User<T, U> {
        name: T,
        y: U,
    }

    
    let user1 = User{name: "Vandam",y:35};
    let user2 = User{name:"James Bond".to_string(),y:"===> 007"};
    // I know it's just a horrible example, just wanted to show you flexibility of generics, 
    // there is no limit, basically especially for data types that just holds data
    println!("User1: {:?} User2: {:?}",user1, user2);


    // => * Generics in Enum Definitions
    // This is a  type of generics we, have seen a lot.

    enum Option<T> { // Same logic, I am gonna refer to my datatype as T
        Some(T), // Some variant will hold whether data types happened to be passed during compile time
        None,
    }

    enum Result<T, E> {
        Ok(T), // if everything went well Ok variant will hold a data of type T
        Err(E), // of something will go wrong Err variant will hold a error of type E

        // And what is important to remember it doesn't really matter what concrete data types will be instead of T and E, appropriate Variants (Ok or Err), will just keep this data, same as with Point<T,U> example.

        // => * Generics in Method Definitions

        // Ok at this point we already know that struct data type, can be used almost like a class in OOP, and for any struct we can implement methods, which will be part of this struct API.
    }
    
    
    struct File<T> {
        name: String,
        data: T,
    }

    impl<T> File<T> { // after keword impl again I need to specify, that I am gonna use T as a placeholder for any datatype I am happen to use
        
        fn new(name: &str, content:T) -> File<T> {
            File { name: String::from(name), data: content }
        }
    }

    let textfile = File::new("lets'go", vec!["K'Maro".to_string()]);
    let imagefile = File::new("MonaLisa",vec![0,123,255]);

    println!("Textfile name {:?}. Textfile content {:?}",textfile.name, textfile.data);
    println!("Imagefile name {:?}. Imagefile content {:?}",imagefile.name, imagefile.data);


    // Generics gonna be your best friend soon along with Traits(next module) and Enums ))
    
    
}
  
    }

WARM-UP Time

Assignment 1:Create a Generic Struct with Methods
Description:
Create a generic struct called Pair that holds two values of the same type. Implement methods to get the first and second values and a method to create a new Pair.

Task:
Define a generic struct Pair<T> with two fields.
Implement a method first that returns the first value.
Implement a method second that returns the second value.
Implement a method new that creates a new Pair.


Assignment 2: Implement a Generic Enum for Handling Custom Errors
Description:
Create a generic enum called CustomResult that can handle custom success and error messages. 
It should be similar to Rust's Result but with your customizations.

Task:
Define a generic enum CustomResult<T, E> with variants Success(T) and Failure(E).
Use this enum to handle custom success and error messages in fn divide_numbers(a: f64, b: f64), keep in mind b can't be zero


Assignment 3: Create a Generic Struct with Associated Methods
Description:
Create a generic struct called Container that holds a collection (Vec) of elements of the same type. 
Implement methods to add elements, remove elements, and retrieve the size of the container.
Task:
Define a generic struct Container<T> that holds a vector of elements of type T.
Implement a method add to add an element to the end of container.
Implement a method remove to remove last added element from the container.
Implement a method size to return the size of the container.
    
    
