class Author:
    def __init__(self,name):
        self.name = name
    def __str__(self):
        return self.name
class Cover:
    def __init__(self, color, materials):
        self.color = color
        self.materials = materials
    def __str__(self):
        return f"{self.materials},{self.color}"
class Publisher:
    def __init__(self, name):
        self.name = name
        self.books = []
    def add_book(self, book):
        self.books.append(book)
    def show_books(self):
        print(f'Издательство: {self.name}')
        for book in self.books:
            print(book)
class Book:
    def __init__(self,  title, author, isbn, year, cover):
        self.author = author
        self.title = title
        self.isbn = isbn
        self.year = year
        self.cover = cover
    def __str__(self):
        return f"{self.author},{self.title},{self.isbn},{self.year},{self.cover}"

class Library:
    def __init__(self):
        self.books = []
    def add_book(self, book):
        self.books.append(book)
    def show_books(self):
        print(f"\n Все книги:")
        for book in self.books:
            print(book)
    def find_by_author(self, author_name):
        print(f"\nПоиск по автору: {author_name}")
        for book in self.books:
            if book.author.name == author_name:
                print(book)

    def find_by_year(self, year):
        print(f"\nПоиск по году: {year}")
        for book in self.books:
            if book.year == year:
                print(book)

author1 = Author("Пушкин")
author2 = Author("Толстой")

cover1 = Cover("Красная", "Твердая")
cover2 = Cover("Синяя", "Мягкая")


book1 = Book("Евгений Онегин", author1, "111", 1833, cover1)
book2 = Book("Война и мир", author2, "222", 1869, cover2)

publisher = Publisher("Русская литература")
publisher.add_book(book1)
publisher.add_book(book2)

publisher.show_books()


library = Library()
library.add_book(book1)
library.add_book(book2)

library.show_books()
library.find_by_author("Пушкин")
library.find_by_year(1869)
