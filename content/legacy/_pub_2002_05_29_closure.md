{
   "categories" : "development",
   "title" : "Achieving Closure",
   "image" : null,
   "date" : "2002-05-29T00:00:00-08:00",
   "tags" : [
      "closure"
   ],
   "thumbnail" : "/images/_pub_2002_05_29_closure/111-achieve_closure.gif",
   "authors" : [
      "simon-cozens"
   ],
   "draft" : null,
   "description" : "Maybe you've heard about closures; they're one of those aspects of Perl - like object-oriented programming - that everyone raves about and you can't really see the big deal until you play around with them and then they just click....",
   "slug" : "/pub/2002/05/29/closure.html"
}



Maybe you've heard about closures; they're one of those aspects of Perl -- like object-oriented programming -- that everyone raves about and you can't really see the big deal until you play around with them and then they just click. In this article, we're going to play around with some closures, in the hope that they'll just click for you.

The nice thing about playing around with closures is that you often don't realize you're doing it. Don't believe me? OK, here's an ordinary piece of Perl:

        my $print_hello = sub { print "Hello, world!"; }

        $print_hello->();

We create a subroutine reference in `$print_hello`, and then we dereference it, calling the subroutine. I suppose we could put that into a subroutine:

        sub make_hello_printer {
            return sub { print "Hello, world!"; }
        }

        my $print_hello = make_hello_printer();
        $print_hello->()

Still nothing magical going on here. And it shouldn't be any surprise to you that we can move the "message" to a separate variable, like this:

        sub make_hello_printer {
            my $message = "Hello, world!";
            return sub { print $message; }
        }

        my $print_hello = make_hello_printer();
        $print_hello->()

As you'd expect, that prints out the `Hello, world!` message. Nothing special going on here, is there? Well, actually, there is. This is a closure. Did you notice?

What's special is that the subroutine reference we created refers to a lexical variable called `$message`. The lexical is defined in `make_hello_printer`, so by rights, it shouldn't be visible outside of `make_hello_printer`, right? We call `make_hello_printer`, `$message` gets created, we return the subroutine reference, and then `$message` goes away, out of scope.

Except it doesn't. When we call our subroutine reference, outside of `make_hello_printer`, it can still see and receive the correct value of `$message`. The subroutine reference forms a *closure*, \`\`enclosing'' the lexical variables it refers to.

Here's the canonical example of closures, that you'll find in practically every Perl book:

        sub make_counter {
            my $start = shift;
            return sub { $start++ }
        }

        my $from_ten = make_counter(10);
        my $from_three = make_counter(3);

        print $from_ten->();       # 10
        print $from_ten->();       # 11
        print $from_three->();     # 3
        print $from_ten->();       # 12
        print $from_three->();     # 4

We've created two "counter" subroutines, which have completely independent values. This happens because each time we call `make_counter`, Perl creates a new lexical for `$start`, which gets wrapped up in the closure we return. So `$from_ten` encloses one `$start` which is initialized to 10, and `$from_three` encloses a **totally different** `$start`, which starts at 3.

It's because of this property that Barrie Slaymaker calls closures "inside-out objects:" objects are data that have some subroutines attached to them, and closures are subroutines that have some data attached to them.

Now, I said that's used in practically every Perl book, because authors try and put off discussing closures until there's little time left and they run out of imagination. (Well, at least that's my excuse ...) However, it's not an entirely practical example, to say the least. So let's try and find a better one.

This example is a bit more complex, but it demonstrates more clearly one extremely useful feature of closures: They can be used to bridge the gap between event-driven programs, which use callbacks extensively, and ordinary procedural code. I recently had to convert a bunch of XML files into an SQL database. Each file constituted a training course, so I wanted to build a data structure that contained the filename plus some of the details I'd parsed from the XML. Here's what I ended up with:

        use XML::Twig;
        my %courses;
        for (<??.xml>) {
            my $name = $_; $name =~ s/.xml//;
            my $t= XML::Twig->new( 
                TwigHandlers => {
                    need => sub { 
                        push @{$courses{$name}{prereqs}}, $_->{'att'}->{course};
                    },
                    # ...
                }
            );
            $t->parsefile($_);
        }

What's going on here? `XML::Twig` is a handy module that can be used to create an XML parser -- these parsers will call "TwigHandlers" when they meet various tags. We go through all the two-letter XML files in the current directory, and create a parser to parse the file. When we see something like this:

        <need course="AA"/>

our `need` handler is called to store the fact that the current course has a prerequisite of the course coded "AA." (`$_->{'att'}->{...}` is `XML::Twig`-speak for "retrieve the value of the attribute called ...")

And that `need` handler is a closure -- it wraps up the name of the current file we're parsing, `$name`, so that it can be referred to whenever `XML::Twig` decides to use it.

There are many other things you can do with closures -- Tom Christiansen once recommended using them for "data hiding" in object-oriented code, since they rely on lexical variables that nothing outside of the closure can see. In fact, some of the most [esoteric and advanced](http://perl.plover.com/lambda/tpj.html) applications of Perl make heavy use of closures.

But as we've seen, some of the most useful uses of closures can happen without you noticing them at all ...
