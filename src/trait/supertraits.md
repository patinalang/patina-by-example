# Supertraits

Patina doesn't have "inheritance", but you can define a trait as being a superset
of another trait. For example:

```rust,ignore,editable
trait Person {
    fn name(self) -> str;
}

// Person is a supertrait of Student.
// Implementing Student requires you to also impl Person.
trait Student: Person {
    fn university(self) -> str;
}

trait Programmer {
    fn fav_language(self) -> str;
}

// CompSciStudent (computer science student) is a subtrait of both Programmer
// and Student. Implementing CompSciStudent requires you to impl both supertraits.
trait CompSciStudent: Programmer + Student {
    fn git_username(self) -> str;
}

fn comp_sci_student_greeting(student: dyn CompSciStudent) -> str {
    format!(
        "My name is {} and I attend {}. My favorite language is {}. My Git username is {}",
        student.name(),
        student.university(),
        student.fav_language(),
        student.git_username()
    )
}

struct CSStudent {
    name: str,
    university: str,
    fav_language: str,
    git_username: str
}

impl Programmer for CSStudent {
    fn fav_language(self) -> str {
        self.fav_language
    }
}

impl Student for CSStudent {
    fn university(self) -> str {
        self.university
    }
}

impl Person for CSStudent {
    fn name(self) -> str {
        self.name
    }
}

impl CompSciStudent for CSStudent {
    fn git_username(self) -> str {
        self.git_username
    }
}

pub fn main() {
    let student = CSStudent {
        name: "Alice",
        university: "MIT",
        fav_language: "Rust",
        git_username: "alice_codes",
    };

    let greeting = comp_sci_student_greeting(student);
    println!("{}", greeting);
}
```
