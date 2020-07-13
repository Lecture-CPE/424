## Binary Files with C++

R.A. Ford 
Department of Math. and Computer Science 
Mount Allison University 
Sackville, NB

### Introduction

บ่อยครั้งเรียก file ว่า streams เพราะข้อมูบันทึกในไฟล์มีลักษณะเป็นลำดับ การอ่านหรือเขียนไฟล์มีการกำหนดจุดเริ่มและจุดสิ้นสุดเป็นลำดับ 


Using streams for file processing is certainly possible in C++, but most C++ textbooks do not include any information regarding the full functionality of streams. This document has been formed to assist students with a background in C++ and data structures with a full description of the C++ stream library. The document is based on the GNU CPP library documentation which at times is not easy to read, especially without examples.
The insertion and extraction operators (i.e. << and >> are meant to be used by programs for writing to and reading from text files; it is assumed that the programmer is familiar with the differences between these two file formats.

In reality there are dozens of extensions with little documentation of ordinary text streams. An additional section will be added to this document at a later time.

Basics of File I/O

Accessing a binary file from a C++ program (by not using the old C functions) requires firstly attaching a stream variable to the file. The usual stream classes ofstream (output file stream) and ifstream (input file stream) are still the types of streams to use. A an additional type called an fstream is provided which allows for files that can be written to and read from if this is a desirable property (in the design of database type programs, this is often the case).
Before any operation can take place on a file, it of course must be opened, and when you are finished with the file, it should be closed to avoid loss of data.

Opening a Stream

The ifstream and ofstream each have member functions named open which are used to attaching the stream to a physical filename and opening the file for either reading or writing. The open member function also provides for a couple of optional arguments that are not often described. The most general prototype of this function is
  void open(const char *filename[, int mode][, int prot]);
The format that I've used indicates that the mode and prot arguments are optional.
The first argument is always the name of the file on the disk that the stream will be attached to. The const modifier is included so that a programmer can write the name of the file (inside double quotes) in the function call. The only tricky part about using the open member function is under DOS based systems (includes Windows) in which directories are separated by a \; recall that the backslash character has a special meaning in C++ strings.

The prot parameter is used to specify the protection (permission) of the file under multiuser operating systems such as Unix. It allows you to specify which users are allowed to look at this file. Under DOS/Windows, this parameter is never used. The mode parameter is usually left out when dealing with text files, but there are some very useful situations under binary files for which this parameter must be set. There are a number of options that can be given for this parameter. If you need to specify more than one of them simply place a vertical bar between them.

ios::in This indicates that the stream will be used for input. This may seem redundant for ifstreams which are automatically marked for input when they are opened, but it has to be used occasionally. When you call open with no second parameter, the parameter is assumed to be ios::in but if you give any other parameter such as ios::binary you will need to specify that the file is an input file as well.
ios::out This indicates that the stream will be used for output. Like ios::in this may seem redundant for ofstreams but for the same reason as above, it usually has to be given.
ios::ate This causes the file pointer to point  at the  end of the file when the file is opened.
ios::trunc This causes the all the existing data in the file to be discarded (erased) when the file is opened. Be very careful not to use this option on a file you do not want destroyed!
ios::binary This causes the file to be accessed as a binary file. Most likely you will need to set this option. If you forget to set this option, many strange problems will occur when reading certain characters like `end of line' and `end of file'.
Example of opening a binary file:
int main()
{
  ifstream infile;
  infile.open("hello.dat", ios::binary | ios::in);
// rest of program

}
Writing to a Binary File

I mentioned once that << is used to write data to a text file. If you had a variable x that contained the value 354 and you used the statment outfile << x; this would cause the character 3, the character 5, and the character 4 to be written (in ASCII form) to the file. This is not binary form which would only require 16-bits. The ofstream class provides a member function named write that allows for information to be written in binary form to the stream. The prototype of the write function is
  ostream& write(void *buffer, streamsize n);
This function causes n bytes to be written from the memory location given by the buffer to the disk and moves the file pointer ahead n bytes.
The parameters types require a little bit of explanation. Even though the return type is ofstream& the return value is usually ignored by most programers. The buffer pointer is of type void this allows for any type of variable to be used as the first parameter. You should not be writing functions with void parameters, this is a very tricky part of programming. The type streamsize is simply a positive integer.

It is rare that you will know exactly how many bytes a particular variable is. To obtain this information, C++ provides a macro (its like a function) named sizeof that takes exactly one parameter and returns the size of the parameter in terms of bytes required for storage.

Below is an example of using the sizeof macro to obtain the size of a variable and writing the contents of a variable to disk. Notice the use of a structure rather than a class; you should not use this method for writing classes to binary files! See the section entitled Writing Classes to Files for a description of how this should be done.

struct Person
{
  char name[50];
  int age;
  char phone[24];
};

int main()
{
  Person me = {"Robert", 28, "364-2534"};
  Person book[30];
  int x = 123;
  double fx = 34.54;
  ofstream outfile;
  outfile.open("junk.dat", ios::binary | ios::out);
  outfile.write(&x, sizeof(int)); // sizeof can take a type
  outfile.write(&fx, sizeof(fx)); // or it can take a variable name
  outfile.write(&me, sizeof(me));
  outfile.write(book, 30*sizeof(Person))
  outfile.close();
}
Reading from a Binary File

Reading data from a binary file is just like writing it except that the function is now called read instead of write When reading data from a file there are a couple of new things to watch out for:
It is the responsibility of the programmer to make sure that the buffer is large enough to hold all the data that is being read. The following code segment would probably result in a crash unless the size of a integer was 7 bytes (unlikely number):
  int main() 
  { 
    int x; 
    ifstream infile; 
    infile.open("silly.dat", ios::binary | ios::in) 
    infile.read(&x, 7); // reads 7 bytes into a cell that is either 2 or 4 
  } 
 
After reading something from the file, the fail() member function should be called to determine if the operation completed succesfully. In C++, no file operations cause the program to stop. If an error occurs and you do not check it, then your program will be running unreliably. See a section further in this document regarding the detection of errors.
File Pointer

 
Whenever data is read from or writen to a file, the data is put or taken from a location inside the file described by the file pointer. In a sequential access file, information is always read from start to end and every time n bytes is read or written, the file pointer is moved n bytes ahead. In a random access file, we are allowed to moved the file pointer to different locations to read data at various locations within a file. Think of a database full of store items. When the item is scanned at the checkout, the barcode is used to look up a description and price of the item. If the file were sequential access, we would have to start searching at the beginning of the file which is probably slower than we would like. This is not a course in file processing, but it suffices to say that if we could move the file pointer directly to the record containing the data we would have to read from the file just once.
The tellp() member function has a prototype of the form

streampos tellp();

This function accepts no parameters, but returns the location given in bytes from the beginning of the file where the file pointer is currently sitting. The next read or write will take place from this location.

The seekp() member function has a prototype of the form

void seekp(streampos location, int relative);

This causes the file pointer to move to another location within the file. The location specifies the number of bytes that will be used to determine the location and the relative parameter indicates whether this is some sort of absolute or relative positioning request. Possible values for relative are: 
 

ios::beg This indicates that the location is the number of bytes from the beginning of the file.
ios::cur This indicates that the location is the number of bytes from the current file pointer location. This allows for a relative positioning of the file pointer.
ios::end This indicates that the location is the number of bytes from the end of the file.
We consider an example that uses both obtaining and setting the file pointer location:
int main() 
{ 
  int x; 
  streampos pos; 
  ifstream infile; 
  infile.open("silly.dat", ios::binary | ios::in); 
  infile.seekp(243, ios::beg); // move 243 bytes into the file 
  infile.read(&x, sizeof(x)); 
  pos = infile.tellg(); 
  cout << "The file pointer is now at location " << pos << endl; 
  infile.seekp(0,ios::end); // seek to the end of the file 
  infile.seekp(-10, ios::cur); // back up 10 bytes 
  infile.close(); 
} 
 

Writing Classes to Binary Files

The easiest way to store records in files is to use astruct If you are keeping track of records in memory structures using classes, then saving these classes to disk takes a little extra work. You cannot simply use a write member function and give the address of the object as the buffer. The reason for this is the presence of member functions. It would not make sense to save the member functions; these member functions end up getting saved as memory locations which would cause your computer to crash if you ever loaded one from disk with an old memory location. It is possible to write objects to disk but it requires that the object have a member function associated with it.
My usual approach is to insert a member function named read and write in each member function. These functions should take an fstream as a parameter as the stream to save itself to. Your program should then open the stream and call the member function with the appropriate stream. The member function should then go through each data field of the object writing them out in a particular order. The read member function must retrieve the information from the disk in exactly the same order.

The example for this section is a little involved, so I've eliminated the non-file member functions. \begin{verbatim} 
#include <iostream.h> 
#include <stdlib.h> 
#include <fstream.h>

class Student 
{ 
  private: 
    int number; 
    char name[50]; 
    float gpa; 
  public: 
    Student(int n, const char *s, float g); 
    void save(ofstream& of); 
    void load(ifstream& inf); 
};

main() 
{ 
  Student me(11321, "Myself", 4.3); 
  ofstream myfile; 
  myfile.open("silly.dat", ios::binary | ios::out); 
  me.save(myfile); 
  myfile.close(); 
  return(0); 
}

void Student::save(ofstream& of) 
{ 
  of.write(&number, sizeof(number)); 
  of.write(name, sizeof(name)); 
  of.write(&gpa, sizeof(gpa)); 
}

void Student::load(ifstream& inf) 
{ 
  inf.read(&number, sizeof(number)); 
  inf.read(name, sizeof(name)); 
  inf.read(&gpa, sizeof(gpa)); 
} 
  
 

What Went Wrong?

In this section, I will point out a couple of methods of determining if a file operation was successful and if not, a couple of methods of determing roughly what went wrong. After every disk operation, a well written program will call the member function fail() to see if the operation completed successfully. It is up to the programmer to determine what should occur when a file operation goes bad. Essentially there are three possibilities: 
 
Ignore the problem and hope it never happens. This is an okay approach to dealing with errors for small programs written to test an idea, but a fully working version of a program should never assume that the user will not make a mistake.
If an error occurs, call exit(EXIT_FAILURE); and have the program terminate. This is slighly better than just hoping it doesn't happen, but in a full version of a program, this could be a real nuisance to the user. Think of what would happen if you spent 5 hours typing an essay, then tried to save it to the T: which did not exist. If your program just aborted then you would have lost 5 hours of work.
When an error occurs, let the user try to correct the error and try the operation again. This is the preferred method as far as the user is concerned, but is usually not trivial to program. You should try to implement this as much as possible.

An unfortunate situation arises when dealing with errors, they are generally physical things which make them operating system dependent. Next, I will list the ANSI (the standard) approach to dealing with errors and the DOS approach to dealing with errors. The ANSI approach is much more general and therefore the error messages will not be precise, but the ANSI approach will work no matter which C++ compiler you use. The DOS error handling eliminates some of the confusion about what happened but obviously is only good on DOS machines that support the library (Turbo C++, Borland C++, and GNU G++ support this library). To make things a little uglier, there appears to be no error support built into streams other than the fail() function. To combat errors we have to rely on some existing C functions which are no problem to use from C++ since C++ is simply an extension of C. 
 

ANSI Errors

ANSI C supports a global variable (oh no, a global variable!) named errno which can be accessed by including errno.h When errors occur the variable is set to a standard error code which should be equivalent on all operating systems. There are too many error codes to bother listing in this document. Usually the best way to discover all error codes is to look at the manual page or on-line help searching on the keyword errno The include file does define a set of constants that can be used to determine the type of error that occurred. For example, error code 22 indicates that the file you just tried to open did not exist. A slightly better way to say 22 is to use the constant ENOENT. There is a function in stdio.h named perror that takes one string as a parameter. When this function is called, the string is displayed on the screen followed by a colon then by a message that describes the value in errno This can be handy if you do not want to write error handlers and just want the program to halt. Below is a simple program that reads a filename from the user, opens the file and displays the fact that the drive was not ready, the file did not exist or the standard error message.
 main() 
{ 
    ifstream data; char filename[50]; 
    cout << "file to open> "; 
    cin.getline(filename, 50); data.open(filename); 
    if (data.fail()) 
        { 
        switch (errno) 
            { 
            case EACCES: 
                // this is set if the drive is not ready in DOS 
                cout << "Drive not ready or permission denied" << endl; 
                break; 
            case ENOENT: 
                cout << "Could not find this file" << endl; 
                break; 
            default: 
                perror("opening data file"); 
        } 
        exit(EXIT_FAILURE); 
        // a real program would then loop back and ask the user to try again. } ...

DOS Extended Errors

If you look at the errors given in the ANSI list you will notice that not many of them are really geared towards DOS; i.e. you don't know for sure if a sector was bad on a disk or the drive door was left open. This is because the ANSI standard was more or less defined on UNIX system where these types of errors are never seen by the users. Most DOS based compilers provide a couple of functions for acessing the DOS extended error which usuallt provides a much more accurate description of the error.