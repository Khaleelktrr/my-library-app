import React, { useState, useEffect, createContext, useContext, useRef } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore, collection, addDoc, onSnapshot, doc, deleteDoc, updateDoc, query, where, getDocs } from 'firebase/firestore';

// Create a context for Firebase and User
const FirebaseContext = createContext(null);

// Custom Message Box Component
const MessageBox = ({ message, onClose }) => {
  if (!message) return null;
  return (
    <div className="fixed inset-0 bg-gray-600 bg-opacity-50 flex items-center justify-center z-50 p-4">
      <div className="bg-white rounded-lg shadow-xl p-6 max-w-sm w-full text-center">
        <p className="text-lg font-semibold mb-4">{message}</p>
        <button
          onClick={onClose}
          className="px-6 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50"
        >
          OK
        </button>
      </div>
    </div>
  );
};

// Custom Confirmation Modal Component
const ConfirmationModal = ({ message, onConfirm, onCancel }) => {
  return (
    <div className="fixed inset-0 bg-gray-600 bg-opacity-50 flex items-center justify-center z-50 p-4">
      <div className="bg-white rounded-lg shadow-xl p-6 max-w-sm w-full text-center">
        <p className="text-lg font-semibold mb-6">{message}</p>
        <div className="flex justify-center space-x-4">
          <button
            onClick={onConfirm}
            className="px-6 py-2 bg-red-600 text-white rounded-md hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-colors duration-300"
          >
            Yes
          </button>
          <button
            onClick={onCancel}
            className="px-6 py-2 bg-gray-300 text-gray-800 rounded-md hover:bg-gray-400 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-opacity-50 transition-colors duration-300"
          >
            No
          </button>
        </div>
      </div>
    </div>
  );
};

// Loading Spinner Component
const LoadingSpinner = () => (
  <div className="flex justify-center items-center h-full">
    <div className="animate-spin rounded-full h-12 w-12 border-b-4 border-blue-500"></div>
  </div>
);

// Header Component (Shared across Admin and Reader views, but with different navigation)
const Header = ({ currentPage, setCurrentPage, userId, isAdminView, setReaderCurrentPage }) => {
  return (
    <header className="bg-gradient-to-r from-blue-600 to-indigo-700 p-4 shadow-lg rounded-b-xl">
      <div className="container mx-auto flex flex-col sm:flex-row justify-between items-center">
        <h1 className="text-3xl font-extrabold text-white mb-2 sm:mb-0">
          Library Management
        </h1>
        <nav className="flex flex-wrap justify-center sm:justify-start space-x-2 sm:space-x-4 mt-2 sm:mt-0">
          {isAdminView ? (
            <>
              <button
                onClick={() => setCurrentPage('add')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'add' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Add Book
              </button>
              <button
                onClick={() => setCurrentPage('list')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'list' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Book List
              </button>
              <button
                onClick={() => setCurrentPage('search')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'search' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Search Book
              </button>
              <button
                onClick={() => setCurrentPage('manageReaders')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'manageReaders' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Manage Readers
              </button>
              <button
                onClick={() => setCurrentPage('borrowReturn')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'borrowReturn' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Borrow/Return
              </button>
              <button
                onClick={() => setCurrentPage('manageCategories')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'manageCategories' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Manage Categories
              </button>
              <button
                onClick={() => setCurrentPage('reports')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'reports' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Reports
              </button>
              <button
                onClick={() => setCurrentPage('notifications')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'notifications' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Notifications
              </button>
              <button
                onClick={() => setCurrentPage('adminReviewsSuggestions')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'adminReviewsSuggestions' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Reviews/Suggestions
              </button>
            </>
          ) : (
            // Reader View Navigation
            <>
              <button
                onClick={() => setReaderCurrentPage('searchBooks')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'searchBooks' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Search Books
              </button>
              <button
                onClick={() => setReaderCurrentPage('borrowedBooksList')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'borrowedBooksList' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Borrowed Books
              </button>
              <button
                onClick={() => setReaderCurrentPage('returnedBooksList')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'returnedBooksList' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Returned Books
              </button>
              <button
                onClick={() => setReaderCurrentPage('readerReviewForm')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'readerReviewForm' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Write Review
              </button>
              <button
                onClick={() => setReaderCurrentPage('readerSuggestBook')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'readerSuggestBook' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Suggest Book
              </button>
              {/* New buttons for Reader View */}
              <button
                onClick={() => setReaderCurrentPage('readerNotifications')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'readerNotifications' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Notifications
              </button>
              <button
                onClick={() => setReaderCurrentPage('readerReports')}
                className={`px-4 py-2 rounded-lg transition-all duration-300 ${
                  currentPage === 'readerReports' ? 'bg-white text-blue-700 shadow-md' : 'text-white hover:bg-blue-500'
                } font-medium`}
              >
                Reports
              </button>
            </>
          )}
        </nav>
      </div>
      {userId && (
        <div className="text-right text-sm text-white mt-2">
          User ID: <span className="font-mono">{userId}</span>
        </div>
      )}
    </header>
  );
};

// Add Book Component
const AddBook = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [book, setBook] = useState({
    title: '',
    author: '',
    isbn: '',
    genre: '', // This will now be the category ID
    price: '',
    language: '',
  });
  const [categories, setCategories] = useState([]);
  const [loading, setLoading] = useState(false);

  // Define available languages
  const languages = ['Kannada', 'Malayalam', 'English', 'Arabic', 'Urdu'];

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }
    const categoriesCollectionRef = collection(db, `artifacts/${userId}/public/data/categories`);
    const unsubscribe = onSnapshot(categoriesCollectionRef, (snapshot) => {
      const categoriesData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      setCategories(categoriesData);
    }, (error) => {
      console.error('Error fetching categories:', error);
      showMessage('Error loading categories.');
    });
    return () => unsubscribe();
  }, [db, auth, userId, isAuthReady, showMessage]);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setBook((prevBook) => ({ ...prevBook, [name]: value }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    if (!book.title || !book.author || !book.isbn || !book.genre || !book.price || !book.language) {
      showMessage('All fields are required: Title, Author, ISBN, Genre, Price, and Language.');
      return;
    }

    setLoading(true);
    try {
      const booksCollectionRef = collection(db, `artifacts/${userId}/public/data/books`);
      const selectedCategory = categories.find(cat => cat.id === book.genre); // Find category name
      await addDoc(booksCollectionRef, {
        ...book,
        genreName: selectedCategory ? selectedCategory.name : book.genre, // Store category name
        isBorrowed: false,
        borrowedBy: null,
        borrowedByName: null,
        borrowedDate: null,
        currentTransactionId: null,
        createdAt: new Date().toISOString(),
        createdBy: userId,
      });
      showMessage('Book added successfully!');
      setBook({
        title: '',
        author: '',
        isbn: '',
        genre: '',
        price: '',
        language: '',
      });
    } catch (error) {
      console.error('Error adding book:', error);
      showMessage('Error adding book.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-2xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Add New Book</h2>
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label htmlFor="title" className="block text-sm font-medium text-gray-700 mb-1">
            Title <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            id="title"
            name="title"
            value={book.title}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          />
        </div>
        <div>
          <label htmlFor="author" className="block text-sm font-medium text-gray-700 mb-1">
            Author <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            id="author"
            name="author"
            value={book.author}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          />
        </div>
        <div>
          <label htmlFor="isbn" className="block text-sm font-medium text-gray-700 mb-1">
            ISBN <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            id="isbn"
            name="isbn"
            value={book.isbn}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          />
        </div>
        <div>
          <label htmlFor="genre" className="block text-sm font-medium text-gray-700 mb-1">
            Genre <span className="text-red-500">*</span>
          </label>
          <select
            id="genre"
            name="genre"
            value={book.genre}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          >
            <option value="">Select a category...</option>
            {categories.map((category) => (
              <option key={category.id} value={category.id}>
                {category.name}
              </option>
            ))}
          </select>
          {categories.length === 0 && (
            <p className="text-red-500 text-xs mt-1">
              No categories available. Please add categories in 'Manage Categories' page.
            </p>
          )}
        </div>
        {/* New fields: Price and Language */}
        <div>
          <label htmlFor="price" className="block text-sm font-medium text-gray-700 mb-1">
            Price <span className="text-red-500">*</span>
          </label>
          <input
            type="number"
            id="price"
            name="price"
            value={book.price}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
            min="0"
            step="0.01"
          />
        </div>
        <div>
          <label htmlFor="language" className="block text-sm font-medium text-gray-700 mb-1">
            Language <span className="text-red-500">*</span>
          </label>
          <select
            id="language"
            name="language"
            value={book.language}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          >
            <option value="">Select a language...</option>
            {languages.map((lang) => (
              <option key={lang} value={lang}>
                {lang}
              </option>
            ))}
          </select>
        </div>
        <button
          type="submit"
          className="w-full bg-blue-600 text-white py-3 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 font-semibold text-lg transition-colors duration-300"
          disabled={loading}
        >
          {loading ? <LoadingSpinner /> : 'Add Book'}
        </button>
      </form>
    </div>
  );
};

// Book List Component
const BookList = ({ showMessage, onEditBook }) => { // Added onEditBook prop
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [allBooks, setAllBooks] = useState([]); // Stores all books from Firestore
  const [filteredBooks, setFilteredBooks] = useState([]); // Books displayed after filtering
  const [loading, setLoading] = useState(true);
  const [searchTerm, setSearchTerm] = useState('');
  const [selectedLanguageFilter, setSelectedLanguageFilter] = useState('');
  const [categories, setCategories] = useState([]); // State for categories
  const [selectedCategoryFilter, setSelectedCategoryFilter] = useState(''); // New state for category filter
  const [showConfirmDeleteModal, setShowConfirmDeleteModal] = useState(false);
  const [bookToDeleteId, setBookToDeleteId] = useState(null);


  // Define available languages for filter
  const languages = ['Kannada', 'Malayalam', 'English', 'Arabic', 'Urdu'];

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    const booksCollectionRef = collection(db, `artifacts/${userId}/public/data/books`);
    const categoriesCollectionRef = collection(db, `artifacts/${userId}/public/data/categories`);

    // Fetch all books
    const unsubscribeBooks = onSnapshot(booksCollectionRef, (snapshot) => {
      const booksData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      // Sort books by creation date (newest first)
      booksData.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
      setAllBooks(booksData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching books:', error);
      showMessage('Error loading books.');
      setLoading(false);
    });

    // Fetch categories
    const unsubscribeCategories = onSnapshot(categoriesCollectionRef, (snapshot) => {
      const categoriesData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      setCategories(categoriesData);
    }, (error) => {
      console.error('Error fetching categories:', error);
      showMessage('Error loading categories.');
    });

    return () => {
      unsubscribeBooks();
      unsubscribeCategories();
    };
  }, [db, auth, userId, isAuthReady, showMessage]);

  // Effect to filter books whenever filters or search term change
  useEffect(() => {
    let currentFilteredBooks = allBooks;

    // Apply search term filter
    if (searchTerm.trim()) {
      const lowerCaseSearchTerm = searchTerm.toLowerCase();
      currentFilteredBooks = currentFilteredBooks.filter(book =>
        book.title.toLowerCase().includes(lowerCaseSearchTerm) ||
        book.author.toLowerCase().includes(lowerCaseSearchTerm) ||
        book.isbn.toLowerCase().includes(lowerCaseSearchTerm) ||
        (book.genreName && book.genreName.toLowerCase().includes(lowerCaseSearchTerm))
      );
    }

    // Apply language filter
    if (selectedLanguageFilter) {
      currentFilteredBooks = currentFilteredBooks.filter(book =>
        book.language === selectedLanguageFilter
      );
    }

    // Apply category filter
    if (selectedCategoryFilter) {
      currentFilteredBooks = currentFilteredBooks.filter(book =>
        book.genre === selectedCategoryFilter // genre stores category ID
      );
    }

    setFilteredBooks(currentFilteredBooks);
  }, [searchTerm, selectedLanguageFilter, selectedCategoryFilter, allBooks]);

  const handleDeleteClick = (id) => {
    setBookToDeleteId(id);
    setShowConfirmDeleteModal(true);
  };

  const confirmDelete = async () => {
    setShowConfirmDeleteModal(false);
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    try {
      const bookDocRef = doc(db, `artifacts/${userId}/public/data/books`, bookToDeleteId);
      await deleteDoc(bookDocRef);
      showMessage('Book deleted successfully!');
    } catch (error) {
      console.error('Error deleting book:', error);
      showMessage('Error deleting book.');
    } finally {
      setBookToDeleteId(null);
    }
  };

  const cancelDelete = () => {
    setShowConfirmDeleteModal(false);
    setBookToDeleteId(null);
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Book List</h2>

      {/* Search and Filter Section */}
      <div className="flex flex-col sm:flex-row gap-4 mb-6">
        <input
          type="text"
          placeholder="Search by title, author, ISBN..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          className="flex-grow px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        />
        <select
          value={selectedLanguageFilter}
          onChange={(e) => setSelectedLanguageFilter(e.target.value)}
          className="px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        >
          <option value="">All Languages</option>
          {languages.map((lang) => (
            <option key={lang} value={lang}>
              {lang}
            </option>
          ))}
        </select>
        <select
          value={selectedCategoryFilter}
          onChange={(e) => setSelectedCategoryFilter(e.target.value)}
          className="px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        >
          <option value="">All Categories</option>
          {categories.map((category) => (
            <option key={category.id} value={category.id}>
              {category.name}
            </option>
          ))}
        </select>
      </div>

      {loading ? (
        <LoadingSpinner />
      ) : filteredBooks.length === 0 ? (
        <p className="text-center text-gray-600">No books available. Please add some books.</p>
      ) : (
        <div className="overflow-x-auto">
          <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
            <thead className="bg-gray-100">
              <tr>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Title</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Author</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">ISBN</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Genre</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Price</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Language</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Status</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Actions</th>
              </tr>
            </thead>
            <tbody>
              {filteredBooks.map((book) => (
                <tr key={book.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                  <td className="py-3 px-4 text-sm text-gray-800">{book.title}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.author}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.isbn}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.genreName || book.genre}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.price ? `₹${parseFloat(book.price).toFixed(2)}` : 'N/A'}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.language || 'N/A'}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">
                    {book.isBorrowed ? (
                      <span className="bg-red-100 text-red-800 text-xs font-medium px-2.5 py-0.5 rounded-full">Borrowed</span>
                    ) : (
                      <span className="bg-green-100 text-green-800 text-xs font-medium px-2.5 py-0.5 rounded-full">Available</span>
                    )}
                  </td>
                  <td className="py-3 px-4 text-sm flex space-x-2">
                    <button
                      onClick={() => onEditBook(book)} // Call onEditBook with the book object
                      className="bg-blue-500 text-white px-3 py-1 rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-colors duration-200"
                    >
                      Edit
                    </button>
                    <button
                      onClick={() => handleDeleteClick(book.id)}
                      className="bg-red-500 text-white px-3 py-1 rounded-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-colors duration-200"
                    >
                      Delete
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
          <p className="text-sm text-gray-500 mt-4">
            Total books: {filteredBooks.length}
          </p>
        </div>
      )}
      {showConfirmDeleteModal && (
        <ConfirmationModal
          message="Are you sure you want to delete this book?"
          onConfirm={confirmDelete}
          onCancel={cancelDelete}
        />
      )}
    </div>
  );
};

// Edit Book Component (Modal/Overlay)
const EditBook = ({ bookData, onClose, showMessage }) => {
  const { db, userId, isAuthReady } = useContext(FirebaseContext);
  const [editedBook, setEditedBook] = useState(bookData);
  const [categories, setCategories] = useState([]);
  const [loading, setLoading] = useState(false);

  // Define available languages
  const languages = ['Kannada', 'Malayalam', 'English', 'Arabic', 'Urdu'];

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }
    const categoriesCollectionRef = collection(db, `artifacts/${userId}/public/data/categories`);
    const unsubscribe = onSnapshot(categoriesCollectionRef, (snapshot) => {
      const categoriesData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      setCategories(categoriesData);
    }, (error) => {
      console.error('Error fetching categories:', error);
      showMessage('Error loading categories.');
    });
    return () => unsubscribe();
  }, [db, userId, isAuthReady, showMessage]);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setEditedBook((prevBook) => ({ ...prevBook, [name]: value }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    if (!editedBook.title || !editedBook.author || !editedBook.isbn || !editedBook.genre || !editedBook.price || !editedBook.language) {
      showMessage('All fields are required: Title, Author, ISBN, Genre, Price, and Language.');
      return;
    }

    setLoading(true);
    try {
      const bookDocRef = doc(db, `artifacts/${userId}/public/data/books`, editedBook.id);
      const selectedCategory = categories.find(cat => cat.id === editedBook.genre);

      await updateDoc(bookDocRef, {
        title: editedBook.title,
        author: editedBook.author,
        isbn: editedBook.isbn,
        genre: editedBook.genre,
        genreName: selectedCategory ? selectedCategory.name : editedBook.genre,
        price: editedBook.price,
        language: editedBook.language,
        updatedAt: new Date().toISOString(),
      });
      showMessage('Book updated successfully!');
      onClose(); // Close the modal after successful update
    } catch (error) {
      console.error('Error updating book:', error);
      showMessage('Error updating book.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="fixed inset-0 bg-gray-600 bg-opacity-75 flex items-center justify-center z-50 p-4 overflow-y-auto">
      <div className="bg-white rounded-lg shadow-xl p-6 max-w-2xl w-full relative">
        <button
          onClick={onClose}
          className="absolute top-4 right-4 text-gray-600 hover:text-gray-900 text-2xl font-bold"
        >
          &times;
        </button>
        <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Edit Book</h2>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label htmlFor="editTitle" className="block text-sm font-medium text-gray-700 mb-1">
              Title <span className="text-red-500">*</span>
            </label>
            <input
              type="text"
              id="editTitle"
              name="title"
              value={editedBook.title}
              onChange={handleChange}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              required
            />
          </div>
          <div>
            <label htmlFor="editAuthor" className="block text-sm font-medium text-gray-700 mb-1">
              Author <span className="text-red-500">*</span>
            </label>
            <input
              type="text"
              id="editAuthor"
              name="author"
              value={editedBook.author}
              onChange={handleChange}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              required
            />
          </div>
          <div>
            <label htmlFor="editIsbn" className="block text-sm font-medium text-gray-700 mb-1">
              ISBN <span className="text-red-500">*</span>
            </label>
            <input
              type="text"
              id="editIsbn"
              name="isbn"
              value={editedBook.isbn}
              onChange={handleChange}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              required
            />
          </div>
          <div>
            <label htmlFor="editGenre" className="block text-sm font-medium text-gray-700 mb-1">
              Genre <span className="text-red-500">*</span>
            </label>
            <select
              id="editGenre"
              name="genre"
              value={editedBook.genre}
              onChange={handleChange}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              required
            >
              <option value="">Select a category...</option>
              {categories.map((category) => (
                <option key={category.id} value={category.id}>
                  {category.name}
                </option>
              ))}
            </select>
            {categories.length === 0 && (
              <p className="text-red-500 text-xs mt-1">
                No categories available. Please add categories in 'Manage Categories' page.
              </p>
            )}
          </div>
          <div>
            <label htmlFor="editPrice" className="block text-sm font-medium text-gray-700 mb-1">
              Price <span className="text-red-500">*</span>
            </label>
            <input
              type="number"
              id="editPrice"
              name="price"
              value={editedBook.price}
              onChange={handleChange}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              required
              min="0"
              step="0.01"
            />
          </div>
          <div>
            <label htmlFor="editLanguage" className="block text-sm font-medium text-gray-700 mb-1">
              Language <span className="text-red-500">*</span>
            </label>
            <select
              id="editLanguage"
              name="language"
              value={editedBook.language}
              onChange={handleChange}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              required
            >
              <option value="">Select a language...</option>
              {languages.map((lang) => (
                <option key={lang} value={lang}>
                  {lang}
                </option>
              ))}
            </select>
          </div>
          <button
            type="submit"
            className="w-full bg-blue-600 text-white py-3 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 font-semibold text-lg transition-colors duration-300"
            disabled={loading}
          >
            {loading ? <LoadingSpinner /> : 'Update Book'}
          </button>
        </form>
      </div>
    </div>
  );
};


// Search Book Component (Admin View)
const SearchBook = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [searchTerm, setSearchTerm] = useState('');
  const [searchResults, setSearchResults] = useState([]);
  const [loading, setLoading] = useState(false);
  const [searchInitiated, setSearchInitiated] = useState(false);
  const [selectedLanguageFilter, setSelectedLanguageFilter] = useState(''); // New state for language filter

  // Define available languages for filter
  const languages = ['Kannada', 'Malayalam', 'English', 'Arabic', 'Urdu'];

  const handleSearch = async (e) => {
    e.preventDefault();
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    if (!searchTerm.trim() && !selectedLanguageFilter) {
      setSearchResults([]);
      setSearchInitiated(false);
      return;
    }

    setLoading(true);
    setSearchInitiated(true);
    try {
      const booksCollectionRef = collection(db, `artifacts/${userId}/public/data/books`);
      const q = query(booksCollectionRef); // Fetch all and filter in memory

      const querySnapshot = await getDocs(q);
      const allBooks = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));

      const filteredBooks = allBooks.filter(book => {
        const matchesSearchTerm =
          book.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
          book.author.toLowerCase().includes(searchTerm.toLowerCase()) ||
          book.isbn.toLowerCase().includes(searchTerm.toLowerCase()) ||
          (book.genreName && book.genreName.toLowerCase().includes(searchTerm.toLowerCase()));

        const matchesLanguage = selectedLanguageFilter ? book.language === selectedLanguageFilter : true;

        return matchesSearchTerm && matchesLanguage;
      });
      setSearchResults(filteredBooks);
    } catch (error) {
      console.error('Error searching books:', error);
      showMessage('Error searching books.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-2xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Search Book</h2>
      <form onSubmit={handleSearch} className="flex flex-col sm:flex-row gap-4 mb-6">
        <input
          type="text"
          placeholder="Search by title, author, ISBN, genre, or language..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          className="flex-grow px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        />
        <select
          value={selectedLanguageFilter}
          onChange={(e) => setSelectedLanguageFilter(e.target.value)}
          className="px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        >
          <option value="">All Languages</option>
          {languages.map((lang) => (
            <option key={lang} value={lang}>
              {lang}
            </option>
          ))}
        </select>
        <button
          type="submit"
          className="bg-blue-600 text-white px-6 py-2 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 font-semibold transition-colors duration-300"
          disabled={loading}
        >
          {loading ? <LoadingSpinner /> : 'Search'}
        </button>
      </form>

      {loading ? (
        <LoadingSpinner />
      ) : searchInitiated && searchResults.length === 0 ? (
        <p className="text-center text-gray-600">No results found.</p>
      ) : searchResults.length > 0 && (
        <div className="overflow-x-auto">
          <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
            <thead className="bg-gray-100">
              <tr>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Title</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Author</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">ISBN</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Genre</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Price</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Language</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Status</th>
              </tr>
            </thead>
            <tbody>
              {searchResults.map((book) => (
                <tr key={book.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                  <td className="py-3 px-4 text-sm text-gray-800">{book.title}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.author}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.isbn}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.genreName || book.genre}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.price ? `₹${parseFloat(book.price).toFixed(2)}` : 'N/A'}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.language || 'N/A'}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">
                    {book.isBorrowed ? (
                      <span className="bg-red-100 text-red-800 text-xs font-medium px-2.5 py-0.5 rounded-full">Borrowed</span>
                    ) : (
                      <span className="bg-green-100 text-green-800 text-xs font-medium px-2.5 py-0.5 rounded-full">Available</span>
                    )}
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      )}
    </div>
  );
};

// Manage Readers Component
const ManageReaders = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [reader, setReader] = useState({
    name: '',
    contact: '',
    membershipId: '',
  });
  const [readers, setReaders] = useState([]);
  const [loading, setLoading] = useState(true);
  const [searchTerm, setSearchTerm] = useState('');
  const [searchResults, setSearchResults] = useState([]);
  const [searchInitiated, setSearchInitiated] = useState(false);
  const [showConfirmDeleteModal, setShowConfirmDeleteModal] = useState(false);
  const [readerToDeleteId, setReaderToDeleteId] = useState(null);

  // Fetch readers on component mount and when auth state changes
  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    const readersCollectionRef = collection(db, `artifacts/${userId}/public/data/readers`);
    const unsubscribe = onSnapshot(readersCollectionRef, (snapshot) => {
      const readersData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      // Sort readers by creation date (newest first)
      readersData.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
      setReaders(readersData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching readers:', error);
      showMessage('Error loading readers.');
      setLoading(false);
    });

    return () => unsubscribe(); // Cleanup on unmount
  }, [db, auth, userId, isAuthReady, showMessage]);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setReader((prevReader) => ({ ...prevReader, [name]: value }));
  };

  const handleAddReader = async (e) => {
    e.preventDefault();
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    if (!reader.name || !reader.membershipId) {
      showMessage('Name and Membership ID are required.');
      return;
    }

    setLoading(true);
    try {
      const readersCollectionRef = collection(db, `artifacts/${userId}/public/data/readers`);
      await addDoc(readersCollectionRef, {
        ...reader,
        createdAt: new Date().toISOString(),
        createdBy: userId,
      });
      showMessage('Reader added successfully!');
      setReader({
        name: '',
        contact: '',
        membershipId: '',
      });
    } catch (error) {
      console.error('Error adding reader:', error);
      showMessage('Error adding reader.');
    } finally {
      setLoading(false);
    }
  };

  const handleDeleteReaderClick = (id) => {
    setReaderToDeleteId(id);
    setShowConfirmDeleteModal(true);
  };

  const confirmDeleteReader = async () => {
    setShowConfirmDeleteModal(false);
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    try {
      const readerDocRef = doc(db, `artifacts/${userId}/public/data/readers`, readerToDeleteId);
      await deleteDoc(readerDocRef);
      showMessage('Reader deleted successfully!');
    } catch (error) {
      console.error('Error deleting reader:', error);
      showMessage('Error deleting reader.');
    } finally {
      setReaderToDeleteId(null);
    }
  };

  const cancelDeleteReader = () => {
    setShowConfirmDeleteModal(false);
    setReaderToDeleteId(null);
  };

  const handleSearchReaders = async (e) => {
    e.preventDefault();
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    if (!searchTerm.trim()) {
      setSearchResults([]);
      setSearchInitiated(false);
      return;
    }

    setLoading(true);
    setSearchInitiated(true);
    try {
      const readersCollectionRef = collection(db, `artifacts/${userId}/public/data/readers`);
      const q = query(readersCollectionRef);

      const querySnapshot = await getDocs(q);
      const allReaders = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));

      const filteredReaders = allReaders.filter(r =>
        r.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
        r.contact.toLowerCase().includes(searchTerm.toLowerCase()) ||
        r.membershipId.toLowerCase().includes(searchTerm.toLowerCase())
      );
      setSearchResults(filteredReaders);
    } catch (error) {
      console.error('Error searching readers:', error);
      showMessage('Error searching readers.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Manage Readers</h2>

      {/* Add Reader Form */}
      <div className="mb-8 p-6 border border-gray-200 rounded-lg bg-gray-50">
        <h3 className="text-xl font-semibold text-gray-700 mb-4">Add New Reader</h3>
        <form onSubmit={handleAddReader} className="space-y-4">
          <div>
            <label htmlFor="readerName" className="block text-sm font-medium text-gray-700 mb-1">
              Name <span className="text-red-500">*</span>
            </label>
            <input
              type="text"
              id="readerName"
              name="name"
              value={reader.name}
              onChange={handleChange}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              required
            />
          </div>
          <div>
            <label htmlFor="readerContact" className="block text-sm font-medium text-gray-700 mb-1">
              Contact
            </label>
            <input
              type="text"
              id="readerContact"
              name="contact"
              value={reader.contact}
              onChange={handleChange}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            />
          </div>
          <div>
            <label htmlFor="membershipId" className="block text-sm font-medium text-gray-700 mb-1">
              Membership ID <span className="text-red-500">*</span>
            </label>
            <input
              type="text"
              id="membershipId"
              name="membershipId"
              value={reader.membershipId}
              onChange={handleChange}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              required
            />
          </div>
          <button
            type="submit"
            className="w-full bg-blue-600 text-white py-3 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 font-semibold text-lg transition-colors duration-300"
            disabled={loading}
          >
            {loading ? <LoadingSpinner /> : 'Add Reader'}
          </button>
        </form>
      </div>

      {/* Search Readers Section */}
      <div className="mb-8 p-6 border border-gray-200 rounded-lg bg-gray-50">
        <h3 className="text-xl font-semibold text-gray-700 mb-4">Search Readers</h3>
        <form onSubmit={handleSearchReaders} className="flex flex-col sm:flex-row gap-4 mb-6">
          <input
            type="text"
            placeholder="Search by name, contact, or membership ID..."
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
            className="flex-grow px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
          />
          <button
            type="submit"
            className="bg-blue-600 text-white px-6 py-2 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 font-semibold transition-colors duration-300"
            disabled={loading}
          >
            {loading ? <LoadingSpinner /> : 'Search'}
          </button>
        </form>

        {loading ? (
          <LoadingSpinner />
        ) : searchInitiated && searchResults.length === 0 ? (
          <p className="text-center text-gray-600">No results found.</p>
        ) : searchResults.length > 0 && (
          <div className="overflow-x-auto">
            <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
              <thead className="bg-gray-100">
                <tr>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Name</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Contact</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Membership ID</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Actions</th>
                </tr>
              </thead>
              <tbody>
                {searchResults.map((r) => (
                  <tr key={r.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                    <td className="py-3 px-4 text-sm text-gray-800">{r.name}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{r.contact || 'N/A'}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{r.membershipId}</td>
                    <td className="py-3 px-4 text-sm">
                      <button
                        onClick={() => handleDeleteReaderClick(r.id)}
                        className="bg-red-500 text-white px-3 py-1 rounded-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-colors duration-200"
                      >
                        Delete
                      </button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
            <p className="text-sm text-gray-500 mt-4">
              Total readers: {searchResults.length}
            </p>
          </div>
        )}
        <h3 className="text-xl font-semibold text-gray-700 mb-4 mt-8">All Readers</h3>
        {loading ? (
          <LoadingSpinner />
        ) : readers.length === 0 ? (
          <p className="text-center text-gray-600">No readers available. Please add some readers.</p>
        ) : (
          <div className="overflow-x-auto">
            <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
              <thead className="bg-gray-100">
                <tr>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Name</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Contact</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Membership ID</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Actions</th>
                </tr>
              </thead>
              <tbody>
                {readers.map((r) => (
                  <tr key={r.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                    <td className="py-3 px-4 text-sm text-gray-800">{r.name}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{r.contact || 'N/A'}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{r.membershipId}</td>
                    <td className="py-3 px-4 text-sm">
                      <button
                        onClick={() => handleDeleteReaderClick(r.id)}
                        className="bg-red-500 text-white px-3 py-1 rounded-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-colors duration-200"
                      >
                        Delete
                      </button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
            <p className="text-sm text-gray-500 mt-4">
              Total readers: {readers.length}
            </p>
          </div>
        )}
        {showConfirmDeleteModal && (
          <ConfirmationModal
            message="Are you sure you want to delete this reader?"
            onConfirm={confirmDeleteReader}
            onCancel={cancelDeleteReader}
          />
        )}
      </div>
    </div>
  );
};

// Searchable Dropdown Component
const SearchableDropdown = ({ options, value, onChange, placeholder, searchTerm, onSearchTermChange, label, displayKey }) => {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef(null);

  const selectedOption = options.find(option => option.id === value);

  useEffect(() => {
    const handleClickOutside = (event) => {
      if (dropdownRef.current && !dropdownRef.current.contains(event.target)) {
        setIsOpen(false);
      }
    };
    document.addEventListener('mousedown', handleClickOutside);
    return () => {
      document.removeEventListener('mousedown', handleClickOutside);
    };
  }, []);

  const handleSelect = (optionId) => {
    onChange(optionId);
    setIsOpen(false);
    onSearchTermChange(''); // Clear search term after selection
  };

  return (
    <div className="relative" ref={dropdownRef}>
      <label className="block text-sm font-medium text-gray-700 mb-1">
        {label} <span className="text-red-500">*</span>
      </label>
      <div className="relative">
        <input
          type="text"
          placeholder={placeholder}
          value={searchTerm || (selectedOption ? selectedOption[displayKey] : '')}
          onChange={(e) => {
            onSearchTermChange(e.target.value);
            setIsOpen(true);
            onChange(''); // Clear selected value when typing
          }}
          onFocus={() => setIsOpen(true)}
          className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500 pr-10"
        />
        <button
          type="button"
          onClick={() => setIsOpen(!isOpen)}
          className="absolute inset-y-0 right-0 flex items-center px-2 text-gray-700 bg-gray-50 rounded-r-md border border-l-0 border-gray-300 hover:bg-gray-100"
        >
          {/* Dropdown arrow icon */}
          <svg className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
            <path fillRule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clipRule="evenodd" />
          </svg>
        </button>
      </div>

      {isOpen && (
        <ul className="absolute z-10 w-full bg-white border border-gray-300 rounded-md shadow-lg max-h-60 overflow-y-auto mt-1">
          {options.length === 0 ? (
            <li className="px-4 py-2 text-gray-500">No results found</li>
          ) : (
            options.map((option) => (
              <li
                key={option.id}
                onClick={() => handleSelect(option.id)}
                className="px-4 py-2 cursor-pointer hover:bg-blue-100 hover:text-blue-700"
              >
                {option[displayKey]}
              </li>
            ))
          )}
        </ul>
      )}
    </div>
  );
};


// Borrow/Return Component
const BorrowReturn = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [allBooks, setAllBooks] = useState([]); // All books from Firestore
  const [allReaders, setAllReaders] = useState([]); // All readers from Firestore
  const [filteredBooks, setFilteredBooks] = useState([]); // Books for dropdown after search
  const [filteredReaders, setFilteredReaders] = useState([]); // Readers for dropdown after search
  const [filteredBorrowedBooks, setFilteredBorrowedBooks] = useState([]); // Borrowed books for table after search

  const [selectedBookId, setSelectedBookId] = useState('');
  const [selectedReaderId, setSelectedReaderId] = useState('');
  const [loading, setLoading] = useState(true);
  const [borrowedBooks, setBorrowedBooks] = useState([]);

  const [bookSearchTerm, setBookSearchTerm] = useState('');
  const [readerSearchTerm, setReaderSearchTerm] = useState('');
  const [returnSearchTerm, setReturnSearchTerm] = useState(''); // New state for return search

  const [showConfirmReturnModal, setShowConfirmReturnModal] = useState(false);
  const [returnTransactionId, setReturnTransactionId] = useState(null);
  const [returnBookId, setReturnBookId] = useState(null);

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    // Fetch books
    const booksCollectionRef = collection(db, `artifacts/${userId}/public/data/books`);
    const unsubscribeBooks = onSnapshot(booksCollectionRef, (snapshot) => {
      const booksData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      setAllBooks(booksData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching books:', error);
      showMessage('Error loading books.');
      setLoading(false);
    });

    // Fetch readers
    const readersCollectionRef = collection(db, `artifacts/${userId}/public/data/readers`);
    const unsubscribeReaders = onSnapshot(readersCollectionRef, (snapshot) => {
      const readersData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      setAllReaders(readersData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching readers:', error);
      showMessage('Error loading readers.');
      setLoading(false);
    });

    // Fetch borrowed books (transactions)
    const transactionsCollectionRef = collection(db, `artifacts/${userId}/public/data/transactions`);
    const qBorrowed = query(transactionsCollectionRef, where('status', '==', 'borrowed'));
    const unsubscribeBorrowed = onSnapshot(qBorrowed, (snapshot) => {
      const borrowedData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setBorrowedBooks(borrowedData);
    }, (error) => {
      console.error('Error fetching borrowed books:', error);
      showMessage('Error loading borrowed books.');
    });


    return () => {
      unsubscribeBooks();
      unsubscribeReaders();
      unsubscribeBorrowed();
    };
  }, [db, auth, userId, isAuthReady, showMessage]);

  // Effect to filter books for dropdown based on search term
  useEffect(() => {
    const lowerCaseSearchTerm = bookSearchTerm.toLowerCase();
    const currentFilteredBooks = allBooks.filter(book =>
      !book.isBorrowed && // Only show available books
      (book.title.toLowerCase().includes(lowerCaseSearchTerm) ||
       book.author.toLowerCase().includes(lowerCaseSearchTerm) ||
       book.isbn.toLowerCase().includes(lowerCaseSearchTerm))
    ).map(book => ({
      id: book.id,
      name: `${book.title} by ${book.author} (ISBN: ${book.isbn})` // Combine fields for display
    }));
    setFilteredBooks(currentFilteredBooks);
  }, [bookSearchTerm, allBooks]);

  // Effect to filter readers for dropdown based on search term
  useEffect(() => {
    const lowerCaseSearchTerm = readerSearchTerm.toLowerCase();
    const currentFilteredReaders = allReaders.filter(reader =>
      reader.name.toLowerCase().includes(lowerCaseSearchTerm) ||
      reader.membershipId.toLowerCase().includes(lowerCaseSearchTerm)
    ).map(reader => ({
      id: reader.id,
      name: `${reader.name} (ID: ${reader.membershipId})` // Combine fields for display
    }));
    setFilteredReaders(currentFilteredReaders);
  }, [readerSearchTerm, allReaders]);

  // Effect to filter borrowed books for the return table based on search term
  useEffect(() => {
    const lowerCaseSearchTerm = returnSearchTerm.toLowerCase();
    const currentFilteredBorrowedBooks = borrowedBooks.filter(transaction =>
      transaction.bookTitle.toLowerCase().includes(lowerCaseSearchTerm) ||
      transaction.readerName.toLowerCase().includes(lowerCaseSearchTerm) ||
      (transaction.bookId && transaction.bookId.toLowerCase().includes(lowerCaseSearchTerm)) || // Search by book ID
      (transaction.readerId && transaction.readerId.toLowerCase().includes(lowerCaseSearchTerm)) // Search by reader ID
    );
    setFilteredBorrowedBooks(currentFilteredBorrowedBooks);
  }, [returnSearchTerm, borrowedBooks]);


  const handleBorrow = async (e) => {
    e.preventDefault();
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    if (!selectedBookId || !selectedReaderId) {
      showMessage('Please select a book and a reader.');
      return;
    }

    const bookToBorrow = allBooks.find(book => book.id === selectedBookId);
    const readerBorrowing = allReaders.find(reader => reader.id === selectedReaderId);

    if (!bookToBorrow || !readerBorrowing) {
      showMessage('Selected book or reader not found.');
      return;
    }
    if (bookToBorrow.isBorrowed) {
      showMessage('This book is already borrowed.');
      return;
    }

    setLoading(true);
    try {
      // 1. Create a new transaction record
      const transactionsCollectionRef = collection(db, `artifacts/${userId}/public/data/transactions`);
      const newTransactionRef = await addDoc(transactionsCollectionRef, {
        bookId: bookToBorrow.id,
        bookTitle: bookToBorrow.title,
        readerId: readerBorrowing.id, // Firestore document ID of the reader
        readerName: readerBorrowing.name,
        borrowedDate: new Date().toISOString(),
        returnDate: null,
        status: 'borrowed',
        createdAt: new Date().toISOString(),
        createdBy: userId, // Admin's UID who performed the action
      });

      // 2. Update the book's status
      const bookDocRef = doc(db, `artifacts/${userId}/public/data/books`, bookToBorrow.id);
      await updateDoc(bookDocRef, {
        isBorrowed: true,
        borrowedBy: readerBorrowing.id, // Store reader's Firestore ID
        borrowedByName: readerBorrowing.name,
        borrowedDate: new Date().toISOString(),
        currentTransactionId: newTransactionRef.id,
      });

      showMessage('Book borrowed successfully!');
      setSelectedBookId('');
      setSelectedReaderId('');
      setBookSearchTerm(''); // Clear search terms after successful borrow
      setReaderSearchTerm('');
    } catch (error) {
      console.error('Error borrowing book:', error);
      showMessage('Error borrowing book.');
    } finally {
      setLoading(false);
    }
  };

  const handleReturnClick = (transactionId, bookId) => {
    setReturnTransactionId(transactionId);
    setReturnBookId(bookId);
    setShowConfirmReturnModal(true);
  };

  const confirmReturn = async () => {
    setShowConfirmReturnModal(false);
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    setLoading(true);
    try {
      // 1. Update the transaction record
      const transactionDocRef = doc(db, `artifacts/${userId}/public/data/transactions`, returnTransactionId);
      await updateDoc(transactionDocRef, {
        returnDate: new Date().toISOString(),
        status: 'returned',
      });

      // 2. Update the book's status
      const bookDocRef = doc(db, `artifacts/${userId}/public/data/books`, returnBookId);
      await updateDoc(bookDocRef, {
        isBorrowed: false,
        borrowedBy: null,
        borrowedByName: null,
        borrowedDate: null,
        currentTransactionId: null,
      });

      showMessage('Book returned successfully!');
    } catch (error) {
      console.error('Error returning book:', error);
      showMessage('Error returning book.');
    } finally {
      setLoading(false);
      setReturnTransactionId(null);
      setReturnBookId(null);
    }
  };

  const cancelReturn = () => {
    setShowConfirmReturnModal(false);
    setReturnTransactionId(null);
    setReturnBookId(null);
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Borrow/Return</h2>

      {/* Borrow Book Section */}
      <div className="mb-8 p-6 border border-gray-200 rounded-lg bg-gray-50">
        <h3 className="text-xl font-semibold text-gray-700 mb-4">Borrow Book</h3>
        <form onSubmit={handleBorrow} className="space-y-4">
          <SearchableDropdown
            label="Select Book"
            options={filteredBooks}
            value={selectedBookId}
            onChange={setSelectedBookId}
            placeholder="Search book..."
            searchTerm={bookSearchTerm}
            onSearchTermChange={setBookSearchTerm}
            displayKey="name" // The key in the option object to display
          />
          <SearchableDropdown
            label="Select Reader"
            options={filteredReaders}
            value={selectedReaderId}
            onChange={setSelectedReaderId}
            placeholder="Search reader..."
            searchTerm={readerSearchTerm}
            onSearchTermChange={setReaderSearchTerm}
            displayKey="name" // The key in the option object to display
          />
          <button
            type="submit"
            className="w-full bg-green-600 text-white py-3 rounded-md hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-opacity-50 font-semibold text-lg transition-colors duration-300"
            disabled={loading}
          >
            {loading ? <LoadingSpinner /> : 'Borrow Book'}
          </button>
        </form>
      </div>

      {/* Currently Borrowed Books Section */}
      <div className="p-6 border border-gray-200 rounded-lg bg-gray-50">
        <h3 className="text-xl font-semibold text-gray-700 mb-4">Currently Borrowed Books</h3>
        <div className="mb-4">
          <label htmlFor="returnSearch" className="block text-sm font-medium text-gray-700 mb-1">
            Search Book to Return:
          </label>
          <input
            type="text"
            id="returnSearch"
            placeholder="Search by book title or reader name..."
            value={returnSearchTerm}
            onChange={(e) => setReturnSearchTerm(e.target.value)}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
          />
        </div>
        {loading ? (
          <LoadingSpinner />
        ) : filteredBorrowedBooks.length === 0 ? (
          <p className="text-center text-gray-600">No books currently borrowed.</p>
        ) : (
          <div className="overflow-x-auto">
            <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
              <thead className="bg-gray-100">
                <tr>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Reader</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Borrowed Date</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Actions</th>
                </tr>
              </thead>
              <tbody>
                {filteredBorrowedBooks.map((transaction) => (
                  <tr key={transaction.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                    <td className="py-3 px-4 text-sm text-gray-800">{transaction.bookTitle}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{transaction.readerName}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">
                      {new Date(transaction.borrowedDate).toLocaleDateString('en-US', {
                        year: 'numeric',
                        month: 'short',
                        day: 'numeric',
                      })}
                    </td>
                    <td className="py-3 px-4 text-sm">
                      <button
                        onClick={() => handleReturnClick(transaction.id, transaction.bookId)}
                        className="bg-purple-600 text-white px-3 py-1 rounded-md hover:bg-purple-700 focus:outline-none focus:ring-2 focus:ring-purple-500 focus:ring-opacity-50 transition-colors duration-200"
                      >
                        Return
                      </button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
            <p className="text-sm text-gray-500 mt-4">
              Total borrowed books: {filteredBorrowedBooks.length}
            </p>
          </div>
        )}
        {showConfirmReturnModal && (
          <ConfirmationModal
            message="Are you sure you want to return this book?"
            onConfirm={confirmReturn}
            onCancel={cancelReturn}
          />
        )}
      </div>
    </div>
  );
};

// Manage Categories Component
const ManageCategories = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [categoryName, setCategoryName] = useState('');
  const [categories, setCategories] = useState([]);
  const [loading, setLoading] = useState(true);
  const [showConfirmDeleteModal, setShowConfirmDeleteModal] = useState(false);
  const [categoryToDeleteId, setCategoryToDeleteId] = useState(null);

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }
    const categoriesCollectionRef = collection(db, `artifacts/${userId}/public/data/categories`);
    const unsubscribe = onSnapshot(categoriesCollectionRef, (snapshot) => {
      const categoriesData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      // Sort categories by creation date (newest first)
      categoriesData.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
      setCategories(categoriesData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching categories:', error);
      showMessage('Error loading categories.');
      setLoading(false);
    });
    return () => unsubscribe();
  }, [db, auth, userId, isAuthReady, showMessage]);

  const handleAddCategory = async (e) => {
    e.preventDefault();
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    if (!categoryName.trim()) {
      showMessage('Category name is required.');
      return;
    }

    setLoading(true);
    try {
      const categoriesCollectionRef = collection(db, `artifacts/${userId}/public/data/categories`);
      await addDoc(categoriesCollectionRef, {
        name: categoryName,
        createdAt: new Date().toISOString(),
        createdBy: userId,
      });
      showMessage('Category added successfully!');
      setCategoryName('');
    } catch (error) {
      console.error('Error adding category:', error);
      showMessage('Error adding category.');
    } finally {
      setLoading(false);
    }
  };

  const handleDeleteCategoryClick = (id) => {
    setCategoryToDeleteId(id);
    setShowConfirmDeleteModal(true);
  };

  const confirmDeleteCategory = async () => {
    setShowConfirmDeleteModal(false);
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    try {
      const categoryDocRef = doc(db, `artifacts/${userId}/public/data/categories`, categoryToDeleteId);
      await deleteDoc(categoryDocRef);
      showMessage('Category deleted successfully!');
    } catch (error) {
      console.error('Error deleting category:', error);
      showMessage('Error deleting category.');
    } finally {
      setCategoryToDeleteId(null);
    }
  };

  const cancelDeleteCategory = () => {
    setShowConfirmDeleteModal(false);
    setCategoryToDeleteId(null);
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-2xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Manage Categories</h2>

      {/* Add Category Form */}
      <div className="mb-8 p-6 border border-gray-200 rounded-lg bg-gray-50">
        <h3 className="text-xl font-semibold text-gray-700 mb-4">Add New Category</h3>
        <form onSubmit={handleAddCategory} className="flex flex-col sm:flex-row gap-4">
          <input
            type="text"
            placeholder="Category Name"
            value={categoryName}
            onChange={(e) => setCategoryName(e.target.value)}
            className="flex-grow px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          />
          <button
            type="submit"
            className="bg-blue-600 text-white px-6 py-2 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 font-semibold transition-colors duration-300"
            disabled={loading}
          >
            {loading ? <LoadingSpinner /> : 'Add Category'}
          </button>
        </form>
      </div>

      {/* Categories List */}
      <h3 className="text-xl font-semibold text-gray-700 mb-4">Available Categories</h3>
      {loading ? (
        <LoadingSpinner />
      ) : categories.length === 0 ? (
        <p className="text-center text-gray-600">No categories available.</p>
      ) : (
        <div className="overflow-x-auto">
          <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
            <thead className="bg-gray-100">
              <tr>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Category Name</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Actions</th>
              </tr>
            </thead>
            <tbody>
              {categories.map((category) => (
                <tr key={category.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                  <td className="py-3 px-4 text-sm text-gray-800">{category.name}</td>
                  <td className="py-3 px-4 text-sm">
                    <button
                      onClick={() => handleDeleteCategoryClick(category.id)}
                      className="bg-red-500 text-white px-3 py-1 rounded-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-colors duration-200"
                    >
                      Delete
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
          <p className="text-sm text-gray-500 mt-4">
            Total categories: {categories.length}
          </p>
        </div>
      )}
      {showConfirmDeleteModal && (
        <ConfirmationModal
          message="Are you sure you want to delete this category?"
          onConfirm={confirmDeleteCategory}
          onCancel={cancelDeleteCategory}
        />
      )}
    </div>
  );
};

// Reader View Components
const ReaderSearchBooks = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [searchTerm, setSearchTerm] = useState('');
  const [allBooks, setAllBooks] = useState([]); // Store all books initially
  const [filteredBooks, setFilteredBooks] = useState([]); // Books displayed after filtering
  const [loading, setLoading] = useState(true);
  const [selectedLanguageFilter, setSelectedLanguageFilter] = useState('');
  const [categories, setCategories] = useState([]); // State for categories
  const [selectedCategoryFilter, setSelectedCategoryFilter] = useState(''); // New state for category filter

  const languages = ['Kannada', 'Malayalam', 'English', 'Arabic', 'Urdu'];

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    const booksCollectionRef = collection(db, `artifacts/${userId}/public/data/books`);
    const categoriesCollectionRef = collection(db, `artifacts/${userId}/public/data/categories`);

    // Fetch all books
    const unsubscribeBooks = onSnapshot(booksCollectionRef, (snapshot) => {
      const booksData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      setAllBooks(booksData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching books:', error);
      showMessage('Error loading books.');
      setLoading(false);
    });

    // Fetch categories
    const unsubscribeCategories = onSnapshot(categoriesCollectionRef, (snapshot) => {
      const categoriesData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      setCategories(categoriesData);
    }, (error) => {
      console.error('Error fetching categories:', error);
      showMessage('Error loading categories.');
    });

    return () => {
      unsubscribeBooks();
      unsubscribeCategories();
    };
  }, [db, auth, userId, isAuthReady, showMessage]);

  // Effect to filter books whenever filters or search term change
  useEffect(() => {
    let currentFilteredBooks = allBooks;

    // Apply search term filter
    if (searchTerm.trim()) {
      const lowerCaseSearchTerm = searchTerm.toLowerCase();
      currentFilteredBooks = currentFilteredBooks.filter(book =>
        book.title.toLowerCase().includes(lowerCaseSearchTerm) ||
        book.author.toLowerCase().includes(lowerCaseSearchTerm) ||
        book.isbn.toLowerCase().includes(lowerCaseSearchTerm) ||
        (book.genreName && book.genreName.toLowerCase().includes(lowerCaseSearchTerm))
      );
    }

    // Apply language filter
    if (selectedLanguageFilter) {
      currentFilteredBooks = currentFilteredBooks.filter(book =>
        book.language === selectedLanguageFilter
      );
    }

    // Apply category filter
    if (selectedCategoryFilter) {
      currentFilteredBooks = currentFilteredBooks.filter(book =>
        book.genre === selectedCategoryFilter // genre stores category ID
      );
    }

    setFilteredBooks(currentFilteredBooks);
  }, [searchTerm, selectedLanguageFilter, selectedCategoryFilter, allBooks]);

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Search Books</h2>
      <div className="flex flex-col sm:flex-row gap-4 mb-6">
        <input
          type="text"
          placeholder="Search by title, author, ISBN, genre, or language..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          className="flex-grow px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        />
        <select
          value={selectedLanguageFilter}
          onChange={(e) => setSelectedLanguageFilter(e.target.value)}
          className="px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        >
          <option value="">All Languages</option>
          {languages.map((lang) => (
            <option key={lang} value={lang}>
              {lang}
            </option>
          ))}
        </select>
        <select
          value={selectedCategoryFilter}
          onChange={(e) => setSelectedCategoryFilter(e.target.value)}
          className="px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        >
          <option value="">All Categories</option>
          {categories.map((category) => (
            <option key={category.id} value={category.id}>
              {category.name}
            </option>
          ))}
        </select>
      </div>

      {loading ? (
        <LoadingSpinner />
      ) : filteredBooks.length === 0 ? (
        <p className="text-center text-gray-600">No results found.</p>
      ) : (
        <div className="overflow-x-auto">
          <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
            <thead className="bg-gray-100">
              <tr>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Title</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Author</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">ISBN</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Genre</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Price</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Language</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Status</th>
              </tr>
            </thead>
            <tbody>
              {filteredBooks.map((book) => (
                <tr key={book.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                  <td className="py-3 px-4 text-sm text-gray-800">{book.title}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.author}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.isbn}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.genreName || book.genre}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.price ? `₹${parseFloat(book.price).toFixed(2)}` : 'N/A'}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{book.language || 'N/A'}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">
                    {book.isBorrowed ? (
                      <span className="bg-red-100 text-red-800 text-xs font-medium px-2.5 py-0.5 rounded-full">
                        Borrowed - {book.borrowedByName || 'N/A'}
                      </span>
                    ) : (
                      <span className="bg-green-100 text-green-800 text-xs font-medium px-2.5 py-0.5 rounded-full">Available</span>
                    )}
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
          <p className="text-sm text-gray-500 mt-4">
            Total books: {filteredBooks.length}
          </p>
        </div>
      )}
    </div>
  );
};

const ReaderBorrowedBooksList = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [borrowedBooks, setBorrowedBooks] = useState([]);
  const [filteredBorrowedBooks, setFilteredBorrowedBooks] = useState([]); // For search results
  const [loading, setLoading] = useState(true);
  const [searchTerm, setSearchTerm] = useState(''); // Search term for borrowed books

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    const transactionsCollectionRef = collection(db, `artifacts/${userId}/public/data/transactions`);
    const q = query(
      transactionsCollectionRef,
      where('status', '==', 'borrowed')
    );

    const unsubscribe = onSnapshot(q, (snapshot) => {
      const borrowedData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setBorrowedBooks(borrowedData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching borrowed books for reader:', error);
      showMessage('Error loading borrowed books.');
      setLoading(false);
    });

    return () => unsubscribe();
  }, [db, auth, userId, isAuthReady, showMessage]);

  // Filter borrowed books based on search term
  useEffect(() => {
    const lowerCaseSearchTerm = searchTerm.toLowerCase();
    const currentFilteredBooks = borrowedBooks.filter(transaction =>
      transaction.bookTitle.toLowerCase().includes(lowerCaseSearchTerm) ||
      transaction.readerName.toLowerCase().includes(lowerCaseSearchTerm) ||
      (transaction.bookId && transaction.bookId.toLowerCase().includes(lowerCaseSearchTerm))
    );
    setFilteredBorrowedBooks(currentFilteredBooks);
  }, [searchTerm, borrowedBooks]);

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Borrowed Books</h2>
      <div className="mb-4">
        <label htmlFor="borrowedSearch" className="block text-sm font-medium text-gray-700 mb-1">
          Search Borrowed Books:
        </label>
        <input
          type="text"
          id="borrowedSearch"
          placeholder="Search by book title or reader name..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        />
      </div>
      {loading ? (
        <LoadingSpinner />
      ) : filteredBorrowedBooks.length === 0 ? (
        <p className="text-center text-gray-600">No books currently borrowed.</p>
      ) : (
        <div className="overflow-x-auto">
          <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
            <thead className="bg-gray-100">
              <tr>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Reader</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Borrowed Date</th>
              </tr>
            </thead>
            <tbody>
              {filteredBorrowedBooks.map((transaction) => (
                <tr key={transaction.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                  <td className="py-3 px-4 text-sm text-gray-800">{transaction.bookTitle}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{transaction.readerName}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">
                    {new Date(transaction.borrowedDate).toLocaleDateString('en-US', {
                      year: 'numeric',
                      month: 'short',
                      day: 'numeric',
                    })}
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
          <p className="text-sm text-gray-500 mt-4">
            Total borrowed books: {filteredBorrowedBooks.length}
          </p>
        </div>
      )}
    </div>
  );
};

// New Component: ReaderReturnedBooksList
const ReaderReturnedBooksList = ({ showMessage }) => {
  const { db, userId, isAuthReady } = useContext(FirebaseContext);
  const [returnedBooks, setReturnedBooks] = useState([]);
  const [filteredReturnedBooks, setFilteredReturnedBooks] = useState([]); // For search results
  const [loading, setLoading] = useState(true);
  const [searchTerm, setSearchTerm] = useState(''); // Search term for returned books

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    const transactionsCollectionRef = collection(db, `artifacts/${userId}/public/data/transactions`);
    const q = query(
      transactionsCollectionRef,
      where('status', '==', 'returned')
    );

    const unsubscribe = onSnapshot(q, (snapshot) => {
      const returnedData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setReturnedBooks(returnedData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching returned books for reader:', error);
      showMessage('Error loading returned books.');
      setLoading(false);
    });

    return () => unsubscribe();
  }, [db, userId, isAuthReady, showMessage]);

  // Filter returned books based on search term
  useEffect(() => {
    const lowerCaseSearchTerm = searchTerm.toLowerCase();
    const currentFilteredBooks = returnedBooks.filter(transaction =>
      transaction.bookTitle.toLowerCase().includes(lowerCaseSearchTerm) ||
      transaction.readerName.toLowerCase().includes(lowerCaseSearchTerm) ||
      (transaction.bookId && transaction.bookId.toLowerCase().includes(lowerCaseSearchTerm))
    );
    setFilteredReturnedBooks(currentFilteredBooks);
  }, [searchTerm, returnedBooks]);

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Returned Books</h2>
      <div className="mb-4">
        <label htmlFor="returnedSearch" className="block text-sm font-medium text-gray-700 mb-1">
          Search Returned Books:
        </label>
        <input
          type="text"
          id="returnedSearch"
          placeholder="Search by book title or reader name..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        />
      </div>
      {loading ? (
        <LoadingSpinner />
      ) : filteredReturnedBooks.length === 0 ? (
        <p className="text-center text-gray-600">No books have been returned yet.</p>
      ) : (
        <div className="overflow-x-auto">
          <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
            <thead className="bg-gray-100">
              <tr>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Reader</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Borrowed Date</th>
                <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Returned Date</th>
              </tr>
            </thead>
            <tbody>
              {filteredReturnedBooks.map((transaction) => (
                <tr key={transaction.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                  <td className="py-3 px-4 text-sm text-gray-800">{transaction.bookTitle}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">{transaction.readerName}</td>
                  <td className="py-3 px-4 text-sm text-gray-800">
                    {new Date(transaction.borrowedDate).toLocaleDateString('en-US', {
                      year: 'numeric',
                      month: 'short',
                      day: 'numeric',
                    })}
                  </td>
                  <td className="py-3 px-4 text-sm text-gray-800">
                    {new Date(transaction.returnDate).toLocaleDateString('en-US', {
                      year: 'numeric',
                      month: 'short',
                      day: 'numeric',
                    })}
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
          <p className="text-sm text-gray-500 mt-4">
            Total returned books: {filteredReturnedBooks.length}
          </p>
        </div>
      )}
    </div>
  );
};


// New Component: Reader Review Form
const ReaderReviewForm = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [review, setReview] = useState({
    bookTitle: '',
    authorName: '',
    reviewText: '',
    reviewerName: '',
    reviewerClass: '',
    reviewerCity: '',
  });
  const [loading, setLoading] = useState(false);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setReview((prevReview) => ({ ...prevReview, [name]: value }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    if (!review.bookTitle || !review.reviewText || !review.reviewerName || !review.reviewerClass || !review.reviewerCity) {
      showMessage('Book Title, Review, Reviewer Name, Class, and City are required.');
      return;
    }

    setLoading(true);
    try {
      const reviewsCollectionRef = collection(db, `artifacts/${userId}/public/data/reviews`);
      await addDoc(reviewsCollectionRef, {
        ...review,
        createdAt: new Date().toISOString(),
        createdBy: userId,
      });
      showMessage('Your review has been submitted successfully!');
      setReview({
        bookTitle: '',
        authorName: '',
        reviewText: '',
        reviewerName: '',
        reviewerClass: '',
        reviewerCity: '',
      });
    } catch (error) {
      console.error('Error submitting review:', error);
      showMessage('Error submitting review.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-2xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Write a Book Review</h2>
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label htmlFor="reviewBookTitle" className="block text-sm font-medium text-gray-700 mb-1">
            Book Title <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            id="reviewBookTitle"
            name="bookTitle"
            value={review.bookTitle}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          />
        </div>
        <div>
          <label htmlFor="reviewAuthorName" className="block text-sm font-medium text-gray-700 mb-1">
            Author Name
          </label>
          <input
            type="text"
            id="reviewAuthorName"
            name="authorName"
            value={review.authorName}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
          />
        </div>
        <div>
          <label htmlFor="reviewText" className="block text-sm font-medium text-gray-700 mb-1">
            Your Review <span className="text-red-500">*</span>
          </label>
          <textarea
            id="reviewText"
            name="reviewText"
            value={review.reviewText}
            onChange={handleChange}
            rows="5"
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          ></textarea>
        </div>
        <div>
          <label htmlFor="reviewerName" className="block text-sm font-medium text-gray-700 mb-1">
            Your Name <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            id="reviewerName"
            name="reviewerName"
            value={review.reviewerName}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          />
        </div>
        <div>
          <label htmlFor="reviewerClass" className="block text-sm font-medium text-gray-700 mb-1">
            Class <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            id="reviewerClass"
            name="reviewerClass"
            value={review.reviewerClass}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          />
        </div>
        <div>
          <label htmlFor="reviewerCity" className="block text-sm font-medium text-gray-700 mb-1">
            City <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            id="reviewerCity"
            name="reviewerCity"
            value={review.reviewerCity}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          />
        </div>
        <button
          type="submit"
          className="w-full bg-blue-600 text-white py-3 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 font-semibold text-lg transition-colors duration-300"
          disabled={loading}
        >
          {loading ? <LoadingSpinner /> : 'Submit Review'}
        </button>
      </form>
    </div>
  );
};

// New Component: Reader Suggest Book
const ReaderSuggestBook = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [suggestion, setSuggestion] = useState({
    bookTitle: '',
    reasonForSuggestion: '',
  });
  const [loading, setLoading] = useState(false);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setSuggestion((prevSuggestion) => ({ ...prevSuggestion, [name]: value }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    if (!suggestion.bookTitle.trim() || !suggestion.reasonForSuggestion.trim()) {
      showMessage('Book Title and Reason for Suggestion are required.');
      return;
    }

    setLoading(true);
    try {
      const suggestionsCollectionRef = collection(db, `artifacts/${userId}/public/data/suggestions`);
      await addDoc(suggestionsCollectionRef, {
        ...suggestion,
        createdAt: new Date().toISOString(),
        createdBy: userId,
      });
      showMessage('Your book suggestion has been submitted successfully!');
      setSuggestion({
        bookTitle: '',
        reasonForSuggestion: '',
      });
    } catch (error) {
      console.error('Error submitting book suggestion:', error);
      showMessage('Error submitting book suggestion.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-2xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Suggest a New Book</h2>
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label htmlFor="suggestBookTitle" className="block text-sm font-medium text-gray-700 mb-1">
            Book Title <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            id="suggestBookTitle"
            name="bookTitle"
            value={suggestion.bookTitle}
            onChange={handleChange}
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          />
        </div>
        <div>
          <label htmlFor="reasonForSuggestion" className="block text-sm font-medium text-gray-700 mb-1">
            Reason for Suggestion <span className="text-red-500">*</span>
          </label>
          <textarea
            id="reasonForSuggestion"
            name="reasonForSuggestion"
            value={suggestion.reasonForSuggestion}
            onChange={handleChange}
            rows="4"
            className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
            required
          ></textarea>
        </div>
        <button
          type="submit"
          className="w-full bg-blue-600 text-white py-3 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 font-semibold text-lg transition-colors duration-300"
          disabled={loading}
        >
          {loading ? <LoadingSpinner /> : 'Submit Suggestion'}
        </button>
      </form>
    </div>
  );
};

// New Component: Admin Reviews and Suggestions
const AdminReviewsAndSuggestions = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [reviews, setReviews] = useState([]);
  const [suggestions, setSuggestions] = useState([]);
  const [loadingReviews, setLoadingReviews] = useState(true);
  const [loadingSuggestions, setLoadingSuggestions] = useState(true);
  const [showConfirmDeleteReviewModal, setShowConfirmDeleteReviewModal] = useState(false);
  const [reviewToDeleteId, setReviewToDeleteId] = useState(null);
  const [showConfirmDeleteSuggestionModal, setShowConfirmDeleteSuggestionModal] = useState(false);
  const [suggestionToDeleteId, setSuggestionToDeleteId] = useState(null);

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    // Fetch Reviews
    const reviewsCollectionRef = collection(db, `artifacts/${userId}/public/data/reviews`);
    const unsubscribeReviews = onSnapshot(reviewsCollectionRef, (snapshot) => {
      const reviewsData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      reviewsData.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
      setReviews(reviewsData);
      setLoadingReviews(false);
    }, (error) => {
      console.error('Error fetching reviews:', error);
      showMessage('Error loading reviews.');
      setLoadingReviews(false);
    });

    // Fetch Suggestions
    const suggestionsCollectionRef = collection(db, `artifacts/${userId}/public/data/suggestions`);
    const unsubscribeSuggestions = onSnapshot(suggestionsCollectionRef, (snapshot) => {
      const suggestionsData = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      suggestionsData.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
      setSuggestions(suggestionsData);
      setLoadingSuggestions(false);
    }, (error) => {
      console.error('Error fetching suggestions:', error);
      showMessage('Error loading suggestions.');
    });

    return () => {
      unsubscribeReviews();
      unsubscribeSuggestions();
    };
  }, [db, auth, userId, isAuthReady, showMessage]);

  const handleDeleteReviewClick = (id) => {
    setReviewToDeleteId(id);
    setShowConfirmDeleteReviewModal(true);
  };

  const confirmDeleteReview = async () => {
    setShowConfirmDeleteReviewModal(false);
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    try {
      const reviewDocRef = doc(db, `artifacts/${userId}/public/data/reviews`, reviewToDeleteId);
      await deleteDoc(reviewDocRef);
      showMessage('Review deleted successfully!');
    } catch (error) {
      console.error('Error deleting review:', error);
      showMessage('Error deleting review.');
    } finally {
      setReviewToDeleteId(null);
    }
  };

  const cancelDeleteReview = () => {
    setShowConfirmDeleteReviewModal(false);
    setReviewToDeleteId(null);
  };

  const handleDeleteSuggestionClick = (id) => {
    setSuggestionToDeleteId(id);
    setShowConfirmDeleteSuggestionModal(true);
  };

  const confirmDeleteSuggestion = async () => {
    setShowConfirmDeleteSuggestionModal(false);
    if (!isAuthReady || !db || !userId) {
      showMessage('Authentication not ready. Please wait.');
      return;
    }
    try {
      const suggestionDocRef = doc(db, `artifacts/${userId}/public/data/suggestions`, suggestionToDeleteId);
      await deleteDoc(suggestionDocRef);
      showMessage('Suggestion deleted successfully!');
    } catch (error) {
      console.error('Error deleting suggestion:', error);
      showMessage('Error deleting suggestion.');
    } finally {
      setSuggestionToDeleteId(null);
    }
  };

  const cancelDeleteSuggestion = () => {
    setShowConfirmDeleteSuggestionModal(false);
    setSuggestionToDeleteId(null);
  };

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-6xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Reviews & Suggestions</h2>

      {/* Reviews Section */}
      <div className="mb-10">
        <h3 className="text-xl font-semibold text-gray-700 mb-4">Book Reviews</h3>
        {loadingReviews ? (
          <LoadingSpinner />
        ) : reviews.length === 0 ? (
          <p className="text-center text-gray-600">No reviews available.</p>
        ) : (
          <div className="overflow-x-auto">
            <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
              <thead className="bg-gray-100">
                <tr>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book Title</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Author</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Review</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Reviewer</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Class</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">City</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Actions</th>
                </tr>
              </thead>
              <tbody>
                {reviews.map((review) => (
                  <tr key={review.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                    <td className="py-3 px-4 text-sm text-gray-800">{review.bookTitle}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{review.authorName || 'N/A'}</td>
                    <td className="py-3 px-4 text-sm text-gray-800 max-w-xs overflow-hidden text-ellipsis whitespace-nowrap">{review.reviewText}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{review.reviewerName}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{review.reviewerClass}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{review.reviewerCity}</td>
                    <td className="py-3 px-4 text-sm">
                      <button
                        onClick={() => handleDeleteReviewClick(review.id)}
                        className="bg-red-500 text-white px-3 py-1 rounded-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-colors duration-200"
                      >
                        Delete
                      </button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
            <p className="text-sm text-gray-500 mt-4">
              Total reviews: {reviews.length}
            </p>
          </div>
        )}
        {showConfirmDeleteReviewModal && (
          <ConfirmationModal
            message="Are you sure you want to delete this review?"
            onConfirm={confirmDeleteReview}
            onCancel={cancelDeleteReview}
          />
        )}
      </div>

      {/* Suggestions Section */}
      <div>
        <h3 className="text-xl font-semibold text-gray-700 mb-4">Book Suggestions</h3>
        {loadingSuggestions ? (
          <LoadingSpinner />
        ) : suggestions.length === 0 ? (
          <p className="text-center text-gray-600">No suggestions available.</p>
        ) : (
          <div className="overflow-x-auto">
            <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
              <thead className="bg-gray-100">
                <tr>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book Title</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Reason for Suggestion</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Actions</th>
                </tr>
              </thead>
              <tbody>
                {suggestions.map((suggestion) => (
                  <tr key={suggestion.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                    <td className="py-3 px-4 text-sm text-gray-800">{suggestion.bookTitle}</td>
                    <td className="py-3 px-4 text-sm text-gray-800 max-w-xs overflow-hidden text-ellipsis whitespace-nowrap">{suggestion.reasonForSuggestion}</td>
                    <td className="py-3 px-4 text-sm">
                      <button
                        onClick={() => handleDeleteSuggestionClick(suggestion.id)}
                        className="bg-red-500 text-white px-3 py-1 rounded-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-colors duration-200"
                      >
                        Delete
                      </button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
            <p className="text-sm text-gray-500 mt-4">
              Total suggestions: {suggestions.length}
            </p>
          </div>
        )}
        {showConfirmDeleteSuggestionModal && (
          <ConfirmationModal
            message="Are you sure you want to delete this suggestion?"
            onConfirm={confirmDeleteSuggestion}
            onCancel={cancelDeleteSuggestion}
          />
        )}
      </div>
    </div>
  );
};

// New Component: Admin Reports
const AdminReports = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [transactions, setTransactions] = useState([]);
  const [loading, setLoading] = useState(true);
  const [reportPeriod, setReportPeriod] = useState('month'); // 'week', 'month', 'year'

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    const transactionsCollectionRef = collection(db, `artifacts/${userId}/public/data/transactions`);
    const unsubscribe = onSnapshot(transactionsCollectionRef, (snapshot) => {
      const transactionsData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setTransactions(transactionsData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching transactions:', error);
      showMessage('Error loading reports.');
      setLoading(false);
    });

    return () => unsubscribe();
  }, [db, auth, userId, isAuthReady, showMessage]);

  const filterTransactionsByPeriod = (data, period) => {
    const now = new Date();
    return data.filter(transaction => {
      const borrowedDate = new Date(transaction.borrowedDate);
      if (period === 'week') {
        const oneWeekAgo = new Date(now.getFullYear(), now.getMonth(), now.getDate() - 7);
        return borrowedDate >= oneWeekAgo && borrowedDate <= now;
      } else if (period === 'month') {
        return borrowedDate.getMonth() === now.getMonth() && borrowedDate.getFullYear() === now.getFullYear();
      } else if (period === 'year') {
        return borrowedDate.getFullYear() === now.getFullYear();
      }
      return true; // No filter
    });
  };

  const getMostActiveReaders = () => {
    const filteredTransactions = filterTransactionsByPeriod(transactions, reportPeriod);
    const readerCounts = {};
    filteredTransactions.forEach(t => {
      if (t.status === 'returned') { // Only count returned books for active readers
        readerCounts[t.readerName] = (readerCounts[t.readerName] || 0) + 1;
      }
    });
    return Object.entries(readerCounts).sort(([, countA], [, countB]) => countB - countA);
  };

  const getMostReadBooks = () => {
    const filteredTransactions = filterTransactionsByPeriod(transactions, reportPeriod);
    const bookCounts = {};
    filteredTransactions.forEach(t => {
      bookCounts[t.bookTitle] = (bookCounts[t.bookTitle] || 0) + 1;
    });
    return Object.entries(bookCounts).sort(([, countA], [, countB]) => countB - countA);
  };

  const activeReaders = getMostActiveReaders();
  const mostReadBooks = getMostReadBooks();

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Reports</h2>

      <div className="mb-6 text-center">
        <label htmlFor="reportPeriod" className="block text-sm font-medium text-gray-700 mb-2">
          Report Period:
        </label>
        <select
          id="reportPeriod"
          value={reportPeriod}
          onChange={(e) => setReportPeriod(e.target.value)}
          className="px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        >
          <option value="week">Week</option>
          <option value="month">Month</option>
          <option value="year">Year</option>
        </select>
      </div>

      {loading ? (
        <LoadingSpinner />
      ) : (
        <>
          {/* Most Active Readers */}
          <div className="mb-8">
            <h3 className="text-xl font-semibold text-gray-700 mb-4">Most Active Readers</h3>
            {activeReaders.length === 0 ? (
              <p className="text-center text-gray-600">No readers found for this period.</p>
            ) : (
              <div className="overflow-x-auto">
                <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
                  <thead className="bg-gray-100">
                    <tr>
                      <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Reader Name</th>
                      <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Books Read</th>
                    </tr>
                  </thead>
                  <tbody>
                    {activeReaders.map(([name, count]) => (
                      <tr key={name} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                        <td className="py-3 px-4 text-sm text-gray-800">{name}</td>
                        <td className="py-3 px-4 text-sm text-gray-800">{count}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}
          </div>

          {/* Most Read Books */}
          <div>
            <h3 className="text-xl font-semibold text-gray-700 mb-4">Most Read Books</h3>
            {mostReadBooks.length === 0 ? (
              <p className="text-center text-gray-600">No books found for this period.</p>
            ) : (
              <div className="overflow-x-auto">
                <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
                  <thead className="bg-gray-100">
                    <tr>
                      <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book Title</th>
                      <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Borrowed Count</th>
                    </tr>
                  </thead>
                  <tbody>
                    {mostReadBooks.map(([title, count]) => (
                      <tr key={title} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                        <td className="py-3 px-4 text-sm text-gray-800">{title}</td>
                        <td className="py-3 px-4 text-sm text-gray-800">{count}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}
          </div>
        </>
      )}
    </div>
  );
};

// New Component: Admin Notifications
const AdminNotifications = ({ showMessage }) => {
  const { db, auth, userId, isAuthReady } = useContext(FirebaseContext);
  const [newBooks, setNewBooks] = useState([]);
  const [overdueBooks, setOverdueBooks] = useState([]);
  const [loadingNewBooks, setLoadingNewBooks] = useState(true);
  const [loadingOverdueBooks, setLoadingOverdueBooks] = useState(true);
  const [selectedMonth, setSelectedMonth] = useState('');

  // Generate month options for the dropdown
  const getMonthOptions = () => {
    const months = [];
    const now = new Date();
    for (let i = 0; i < 12; i++) { // Last 12 months
      const date = new Date(now.getFullYear(), now.getMonth() - i, 1);
      months.push({
        value: `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`,
        label: date.toLocaleDateString('en-US', { year: 'numeric', month: 'long' }),
      });
    }
    return months;
  };

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    // Fetch new books based on selected month
    const fetchNewBooks = async () => {
      setLoadingNewBooks(true);
      try {
        const booksCollectionRef = collection(db, `artifacts/${userId}/public/data/books`);
        const querySnapshot = await getDocs(query(booksCollectionRef));
        const allBooks = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));

        const filteredNewBooks = allBooks.filter(book => {
          if (!book.createdAt) return false;
          const bookCreationDate = new Date(book.createdAt);
          if (selectedMonth) {
            const [year, month] = selectedMonth.split('-').map(Number);
            return bookCreationDate.getFullYear() === year && (bookCreationDate.getMonth() + 1) === month;
          } else {
            // Default to last month if no specific month is selected
            const now = new Date();
            const lastMonth = new Date(now.getFullYear(), now.getMonth() - 1, 1);
            return bookCreationDate >= lastMonth && bookCreationDate <= now;
          }
        });
        filteredNewBooks.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
        setNewBooks(filteredNewBooks);
      } catch (error) {
        console.error('Error fetching new books:', error);
        showMessage('Error loading new books.');
      } finally {
        setLoadingNewBooks(false);
      }
    };

    // Fetch overdue books
    const fetchOverdueBooks = async () => {
      setLoadingOverdueBooks(true);
      try {
        const transactionsCollectionRef = collection(db, `artifacts/${userId}/public/data/transactions`);
        const q = query(transactionsCollectionRef, where('status', '==', 'borrowed'));
        const querySnapshot = await getDocs(q);
        const borrowedTransactions = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));

        const now = new Date();
        const overdueThreshold = 10 * 24 * 60 * 60 * 1000; // 10 days in milliseconds

        const overdue = borrowedTransactions.filter(t => {
          if (!t.borrowedDate) return false;
          const borrowedTime = new Date(t.borrowedDate).getTime();
          return (now.getTime() - borrowedTime) > overdueThreshold;
        });
        setOverdueBooks(overdue);
      } catch (error) {
        console.error('Error fetching overdue books:', error);
        showMessage('Error loading overdue books.');
      } finally {
        setLoadingOverdueBooks(false);
      }
    };

    fetchNewBooks();
    fetchOverdueBooks();

  }, [db, auth, userId, isAuthReady, showMessage, selectedMonth]);

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Notifications</h2>

      {/* New Books Section */}
      <div className="mb-10">
        <h3 className="text-xl font-semibold text-gray-700 mb-4">New Books</h3>
        <div className="mb-4">
          <label htmlFor="monthSelect" className="block text-sm font-medium text-gray-700 mb-1">
            Select Month:
          </label>
          <select
            id="monthSelect"
            value={selectedMonth}
            onChange={(e) => setSelectedMonth(e.target.value)}
            className="px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
          >
            <option value="">Last Month</option>
            {getMonthOptions().map(option => (
              <option key={option.value} value={option.value}>{option.label}</option>
            ))}
          </select>
        </div>
        {loadingNewBooks ? (
          <LoadingSpinner />
        ) : newBooks.length === 0 ? (
          <p className="text-center text-gray-600">No new books available for this period.</p>
        ) : (
          <div className="overflow-x-auto">
            <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
              <thead className="bg-gray-100">
                <tr>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Title</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Author</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Added Date</th>
                </tr>
              </thead>
              <tbody>
                {newBooks.map((book) => (
                  <tr key={book.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                    <td className="py-3 px-4 text-sm text-gray-800">{book.title}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{book.author}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">
                      {new Date(book.createdAt).toLocaleDateString('en-US', {
                        year: 'numeric',
                        month: 'short',
                        day: 'numeric',
                      })}
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
            <p className="text-sm text-gray-500 mt-4">
              Total new books: {newBooks.length}
            </p>
          </div>
        )}
      </div>

      {/* Overdue Books Section */}
      <div>
        <h3 className="text-xl font-semibold text-gray-700 mb-4">Overdue Books</h3>
        <p className="text-sm text-gray-600 mb-4">
          According to library rules, books must be returned within 10 days.
        </p>
        {loadingOverdueBooks ? (
          <LoadingSpinner />
        ) : overdueBooks.length === 0 ? (
          <p className="text-center text-gray-600">No overdue books found.</p>
        ) : (
          <div className="overflow-x-auto">
            <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
              <thead className="bg-gray-100">
                <tr>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Reader Name</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book Title</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Borrowed Date</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Overdue Days</th>
                </tr>
              </thead>
              <tbody>
                {overdueBooks.map((transaction) => {
                  const borrowedDate = new Date(transaction.borrowedDate);
                  const now = new Date();
                  const diffTime = Math.abs(now.getTime() - borrowedDate.getTime());
                  const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
                  const overdueDays = diffDays > 10 ? diffDays - 10 : 0; // Calculate days overdue

                  return (
                    <tr key={transaction.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                      <td className="py-3 px-4 text-sm text-gray-800">{transaction.readerName}</td>
                      <td className="py-3 px-4 text-sm text-gray-800">{transaction.bookTitle}</td>
                      <td className="py-3 px-4 text-sm text-gray-800">
                        {borrowedDate.toLocaleDateString('en-US', {
                          year: 'numeric',
                          month: 'short',
                          day: 'numeric',
                        })}
                      </td>
                      <td className="py-3 px-4 text-sm text-red-700 font-semibold">{overdueDays}</td>
                    </tr>
                  );
                })}
              </tbody>
            </table>
            <p className="text-sm text-gray-500 mt-4">
              Total overdue books: {overdueBooks.length}
            </p>
          </div>
        )}
      </div>
    </div>
  );
};

// New Component: Reader Notifications (Simplified version of AdminNotifications)
const ReaderNotifications = ({ showMessage }) => {
  const { db, userId, isAuthReady } = useContext(FirebaseContext);
  const [newBooks, setNewBooks] = useState([]);
  const [overdueBooks, setOverdueBooks] = useState([]);
  const [loadingNewBooks, setLoadingNewBooks] = useState(true);
  const [loadingOverdueBooks, setLoadingOverdueBooks] = useState(true);
  const [selectedMonth, setSelectedMonth] = useState('');

  const getMonthOptions = () => {
    const months = [];
    const now = new Date();
    for (let i = 0; i < 12; i++) {
      const date = new Date(now.getFullYear(), now.getMonth() - i, 1);
      months.push({
        value: `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`,
        label: date.toLocaleDateString('en-US', { year: 'numeric', month: 'long' }),
      });
    }
    return months;
  };

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    const fetchNewBooks = async () => {
      setLoadingNewBooks(true);
      try {
        const booksCollectionRef = collection(db, `artifacts/${userId}/public/data/books`);
        const querySnapshot = await getDocs(query(booksCollectionRef));
        const allBooks = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));

        const filteredNewBooks = allBooks.filter(book => {
          if (!book.createdAt) return false;
          const bookCreationDate = new Date(book.createdAt);
          if (selectedMonth) {
            const [year, month] = selectedMonth.split('-').map(Number);
            return bookCreationDate.getFullYear() === year && (bookCreationDate.getMonth() + 1) === month;
          } else {
            const now = new Date();
            const lastMonth = new Date(now.getFullYear(), now.getMonth() - 1, 1);
            return bookCreationDate >= lastMonth && bookCreationDate <= now;
          }
        });
        filteredNewBooks.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
        setNewBooks(filteredNewBooks);
      } catch (error) {
        console.error('Error fetching new books:', error);
        showMessage('Error loading new books.');
      } finally {
        setLoadingNewBooks(false);
      }
    };

    const fetchOverdueBooks = async () => {
      setLoadingOverdueBooks(true);
      try {
        const transactionsCollectionRef = collection(db, `artifacts/${userId}/public/data/transactions`);
        const q = query(transactionsCollectionRef, where('status', '==', 'borrowed'));
        const querySnapshot = await getDocs(q);
        const borrowedTransactions = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));

        const now = new Date();
        const overdueThreshold = 10 * 24 * 60 * 60 * 1000;

        const overdue = borrowedTransactions.filter(t => {
          if (!t.borrowedDate) return false;
          const borrowedTime = new Date(t.borrowedDate).getTime();
          return (now.getTime() - borrowedTime) > overdueThreshold;
        });
        setOverdueBooks(overdue);
      } catch (error) {
        console.error('Error fetching overdue books:', error);
        showMessage('Error loading overdue books.');
      } finally {
        setLoadingOverdueBooks(false);
      }
    };

    fetchNewBooks();
    fetchOverdueBooks();

  }, [db, userId, isAuthReady, showMessage, selectedMonth]);

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Notifications</h2>

      {/* New Books Section */}
      <div className="mb-10">
        <h3 className="text-xl font-semibold text-gray-700 mb-4">New Books</h3>
        <div className="mb-4">
          <label htmlFor="monthSelect" className="block text-sm font-medium text-gray-700 mb-1">
            Select Month:
          </label>
          <select
            id="monthSelect"
            value={selectedMonth}
            onChange={(e) => setSelectedMonth(e.target.value)}
            className="px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
          >
            <option value="">Last Month</option>
            {getMonthOptions().map(option => (
              <option key={option.value} value={option.value}>{option.label}</option>
            ))}
          </select>
        </div>
        {loadingNewBooks ? (
          <LoadingSpinner />
        ) : newBooks.length === 0 ? (
          <p className="text-center text-gray-600">No new books available for this period.</p>
        ) : (
          <div className="overflow-x-auto">
            <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
              <thead className="bg-gray-100">
                <tr>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Title</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Author</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Added Date</th>
                </tr>
              </thead>
              <tbody>
                {newBooks.map((book) => (
                  <tr key={book.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                    <td className="py-3 px-4 text-sm text-gray-800">{book.title}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">{book.author}</td>
                    <td className="py-3 px-4 text-sm text-gray-800">
                      {new Date(book.createdAt).toLocaleDateString('en-US', {
                        year: 'numeric',
                        month: 'short',
                        day: 'numeric',
                      })}
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
            <p className="text-sm text-gray-500 mt-4">
              Total new books: {newBooks.length}
            </p>
          </div>
        )}
      </div>

      {/* Overdue Books Section */}
      <div>
        <h3 className="text-xl font-semibold text-gray-700 mb-4">Overdue Books</h3>
        <p className="text-sm text-gray-600 mb-4">
          According to library rules, books must be returned within 10 days.
        </p>
        {loadingOverdueBooks ? (
          <LoadingSpinner />
        ) : overdueBooks.length === 0 ? (
          <p className="text-center text-gray-600">No overdue books found.</p>
        ) : (
          <div className="overflow-x-auto">
            <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
              <thead className="bg-gray-100">
                <tr>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Reader Name</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book Title</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Borrowed Date</th>
                  <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Overdue Days</th>
                </tr>
              </thead>
              <tbody>
                {overdueBooks.map((transaction) => {
                  const borrowedDate = new Date(transaction.borrowedDate);
                  const now = new Date();
                  const diffTime = Math.abs(now.getTime() - borrowedDate.getTime());
                  const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
                  const overdueDays = diffDays > 10 ? diffDays - 10 : 0; // Calculate days overdue

                  return (
                    <tr key={transaction.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                      <td className="py-3 px-4 text-sm text-gray-800">{transaction.readerName}</td>
                      <td className="py-3 px-4 text-sm text-gray-800">{transaction.bookTitle}</td>
                      <td className="py-3 px-4 text-sm text-gray-800">
                        {borrowedDate.toLocaleDateString('en-US', {
                          year: 'numeric',
                          month: 'short',
                          day: 'numeric',
                        })}
                      </td>
                      <td className="py-3 px-4 text-sm text-red-700 font-semibold">{overdueDays}</td>
                    </tr>
                  );
                })}
              </tbody>
            </table>
            <p className="text-sm text-gray-500 mt-4">
              Total overdue books: {overdueBooks.length}
            </p>
          </div>
        )}
      </div>
    </div>
  );
};

// New Component: Reader Reports (Simplified version of AdminReports)
const ReaderReports = ({ showMessage }) => {
  const { db, userId, isAuthReady } = useContext(FirebaseContext);
  const [transactions, setTransactions] = useState([]);
  const [loading, setLoading] = useState(true);
  const [reportPeriod, setReportPeriod] = useState('month'); // 'week', 'month', 'year'

  useEffect(() => {
    if (!isAuthReady || !db || !userId) {
      return;
    }

    const transactionsCollectionRef = collection(db, `artifacts/${userId}/public/data/transactions`);
    const unsubscribe = onSnapshot(transactionsCollectionRef, (snapshot) => {
      const transactionsData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setTransactions(transactionsData);
      setLoading(false);
    }, (error) => {
      console.error('Error fetching transactions:', error);
      showMessage('Error loading reports.');
      setLoading(false);
    });

    return () => unsubscribe();
  }, [db, userId, isAuthReady, showMessage]);

  const filterTransactionsByPeriod = (data, period) => {
    const now = new Date();
    return data.filter(transaction => {
      const transactionDate = new Date(transaction.borrowedDate || transaction.returnDate); // Use either date
      if (period === 'week') {
        const oneWeekAgo = new Date(now.getFullYear(), now.getMonth(), now.getDate() - 7);
        return transactionDate >= oneWeekAgo && transactionDate <= now;
      } else if (period === 'month') {
        return transactionDate.getMonth() === now.getMonth() && transactionDate.getFullYear() === now.getFullYear();
      } else if (period === 'year') {
        return transactionDate.getFullYear() === now.getFullYear();
      }
      return true; // No filter
    });
  };

  const getMostReadBooks = () => {
    const filteredTransactions = filterTransactionsByPeriod(transactions, reportPeriod);
    const bookCounts = {};
    filteredTransactions.forEach(t => {
      bookCounts[t.bookTitle] = (bookCounts[t.bookTitle] || 0) + 1;
    });
    return Object.entries(bookCounts).sort(([, countA], [, countB]) => countB - countA);
  };

  const getAllReturnedBooks = () => {
    const filteredTransactions = filterTransactionsByPeriod(transactions, reportPeriod);
    return filteredTransactions.filter(t => t.status === 'returned').sort((a, b) => new Date(b.returnDate) - new Date(a.returnDate));
  };

  const mostReadBooks = getMostReadBooks();
  const returnedBooks = getAllReturnedBooks();

  return (
    <div className="bg-white p-8 rounded-xl shadow-lg max-w-4xl mx-auto mt-8 border border-gray-200">
      <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Reports</h2>

      <div className="mb-6 text-center">
        <label htmlFor="reportPeriod" className="block text-sm font-medium text-gray-700 mb-2">
          Report Period:
        </label>
        <select
          id="reportPeriod"
          value={reportPeriod}
          onChange={(e) => setReportPeriod(e.target.value)}
          className="px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        >
          <option value="week">Week</option>
          <option value="month">Month</option>
          <option value="year">Year</option>
          <option value="all">All Time</option>
        </select>
      </div>

      {loading ? (
        <LoadingSpinner />
      ) : (
        <>
          {/* Most Read Books */}
          <div className="mb-8">
            <h3 className="text-xl font-semibold text-gray-700 mb-4">Most Read Books</h3>
            {mostReadBooks.length === 0 ? (
              <p className="text-center text-gray-600">No books found for this period.</p>
            ) : (
              <div className="overflow-x-auto">
                <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
                  <thead className="bg-gray-100">
                    <tr>
                      <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book Title</th>
                      <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Borrowed Count</th>
                    </tr>
                  </thead>
                  <tbody>
                    {mostReadBooks.map(([title, count]) => (
                      <tr key={title} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                        <td className="py-3 px-4 text-sm text-gray-800">{title}</td>
                        <td className="py-3 px-4 text-sm text-gray-800">{count}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}
          </div>

          {/* All Returned Books */}
          <div>
            <h3 className="text-xl font-semibold text-gray-700 mb-4">Returned Books</h3>
            {returnedBooks.length === 0 ? (
              <p className="text-center text-gray-600">No books returned for this period.</p>
            ) : (
              <div className="overflow-x-auto">
                <table className="min-w-full bg-white border border-gray-300 rounded-lg overflow-hidden">
                  <thead className="bg-gray-100">
                    <tr>
                      <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Book</th>
                      <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Reader</th>
                      <th className="py-3 px-4 text-left text-sm font-semibold text-gray-700">Returned Date</th>
                    </tr>
                  </thead>
                  <tbody>
                    {returnedBooks.map((transaction) => (
                      <tr key={transaction.id} className="border-b border-gray-200 hover:bg-gray-50 transition-colors duration-150">
                        <td className="py-3 px-4 text-sm text-gray-800">{transaction.bookTitle}</td>
                        <td className="py-3 px-4 text-sm text-gray-800">{transaction.readerName}</td>
                        <td className="py-3 px-4 text-sm text-gray-800">
                          {new Date(transaction.returnDate).toLocaleDateString('en-US', {
                            year: 'numeric',
                            month: 'short',
                            day: 'numeric',
                          })}
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}
          </div>
        </>
      )}
    </div>
  );
};

// Admin Password Modal Component
const AdminPasswordModal = ({ onAuthenticate, onCancel, showMessage }) => {
  const [password, setPassword] = useState('');
  const ADMIN_PASSWORD = 'admin123'; // Hardcoded admin password for demonstration

  const handleSubmit = (e) => {
    e.preventDefault();
    if (password === ADMIN_PASSWORD) {
      onAuthenticate();
    } else {
      showMessage('Incorrect password. Please try again.');
      setPassword(''); // Clear password field on incorrect attempt
    }
  };

  return (
    <div className="fixed inset-0 bg-gray-600 bg-opacity-75 flex items-center justify-center z-50 p-4">
      <div className="bg-white rounded-lg shadow-xl p-6 max-w-sm w-full relative">
        <h2 className="text-2xl font-bold text-gray-800 mb-6 text-center">Admin Access</h2>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label htmlFor="adminPassword" className="block text-sm font-medium text-gray-700 mb-1">
              Enter Password:
            </label>
            <input
              type="password"
              id="adminPassword"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              className="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
              required
              autoFocus
            />
          </div>
          <div className="flex justify-end space-x-3">
            <button
              type="button"
              onClick={onCancel}
              className="px-4 py-2 bg-gray-300 text-gray-800 rounded-md hover:bg-gray-400 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-opacity-50 transition-colors duration-300"
            >
              Cancel
            </button>
            <button
              type="submit"
              className="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 font-semibold transition-colors duration-300"
            >
              Access
            </button>
          </div>
        </form>
      </div>
    </div>
  );
};


// Main App Component
export default function App() {
  const [db, setDb] = useState(null);
  const [auth, setAuth] = useState(null);
  const [userId, setUserId] = useState(null);
  const [isAuthReady, setIsAuthReady] = useState(false);
  const [isAdminView, setIsAdminView] = useState(false); // Default to Reader view
  const [currentPage, setCurrentPage] = useState('list'); // Default admin page
  const [readerCurrentPage, setReaderCurrentPage] = useState('searchBooks'); // Default reader page
  const [message, setMessage] = useState('');
  const [editingBook, setEditingBook] = useState(null); // State to hold book being edited
  const [showAdminPasswordModal, setShowAdminPasswordModal] = useState(false); // State for password modal

  // Function to display a message
  const showMessage = (msg) => {
    setMessage(msg);
  };

  // Function to close the message box
  const closeMessage = () => {
    setMessage('');
  };

  useEffect(() => {
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
    const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
    const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

    try {
      const app = initializeApp(firebaseConfig);
      const firestore = getFirestore(app);
      const firebaseAuth = getAuth(app);

      setDb(firestore);
      setAuth(firebaseAuth);

      const unsubscribe = onAuthStateChanged(firebaseAuth, async (user) => {
        if (user) {
          setUserId(user.uid);
        } else {
          // Sign in anonymously if no user is authenticated
          try {
            if (initialAuthToken) {
              await signInWithCustomToken(firebaseAuth, initialAuthToken);
            } else {
              await signInAnonymously(firebaseAuth);
            }
          } catch (error) {
            console.error('Firebase authentication error:', error);
            showMessage(`Authentication failed: ${error.message}`);
          }
        }
        setIsAuthReady(true);
      });

      return () => unsubscribe();
    } catch (e) {
      console.error('Error initializing Firebase:', e);
      showMessage(`Error initializing Firebase: ${e.message}`);
    }
  }, []);

  const handleAdminViewClick = () => {
    setShowAdminPasswordModal(true);
  };

  const handleAuthenticateAdmin = () => {
    setIsAdminView(true);
    setCurrentPage('list'); // Reset to default admin page
    setShowAdminPasswordModal(false);
  };

  const handleCancelAdminAccess = () => {
    setShowAdminPasswordModal(false);
    setIsAdminView(false); // Ensure it stays in reader view if cancelled
    setReaderCurrentPage('searchBooks'); // Reset to default reader page
  };


  const renderPage = () => {
    if (isAdminView) {
      switch (currentPage) {
        case 'add':
          return <AddBook showMessage={showMessage} />;
        case 'list':
          return <BookList showMessage={showMessage} onEditBook={setEditingBook} />; // Pass setEditingBook
        case 'search':
          return <SearchBook showMessage={showMessage} />;
        case 'manageReaders':
          return <ManageReaders showMessage={showMessage} />;
        case 'borrowReturn':
          return <BorrowReturn showMessage={showMessage} />;
        case 'manageCategories':
          return <ManageCategories showMessage={showMessage} />;
        case 'reports':
          return <AdminReports showMessage={showMessage} />;
        case 'notifications':
          return <AdminNotifications showMessage={showMessage} />;
        case 'adminReviewsSuggestions':
          return <AdminReviewsAndSuggestions showMessage={showMessage} />;
        default:
          return <BookList showMessage={showMessage} onEditBook={setEditingBook} />;
      }
    } else {
      switch (readerCurrentPage) {
        case 'searchBooks':
          return <ReaderSearchBooks showMessage={showMessage} />;
        case 'borrowedBooksList':
          return <ReaderBorrowedBooksList showMessage={showMessage} />;
        case 'returnedBooksList':
          return <ReaderReturnedBooksList showMessage={showMessage} />; // New: Render Returned Books List
        case 'readerReviewForm':
          return <ReaderReviewForm showMessage={showMessage} />;
        case 'readerSuggestBook':
          return <ReaderSuggestBook showMessage={showMessage} />;
        case 'readerNotifications':
          return <ReaderNotifications showMessage={showMessage} />; // New Reader Notifications
        case 'readerReports':
          return <ReaderReports showMessage={showMessage} />; // New Reader Reports
        default:
          return <ReaderSearchBooks showMessage={showMessage} />;
      }
    }
  };

  return (
    <FirebaseContext.Provider value={{ db, auth, userId, isAuthReady }}>
      <div className="min-h-screen bg-gray-100 font-sans antialiased">
        <style>{`
          @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap');
          body {
            font-family: 'Inter', sans-serif;
          }
        `}</style>
        <script src="https://cdn.tailwindcss.com"></script>

        <Header
          currentPage={isAdminView ? currentPage : readerCurrentPage}
          setCurrentPage={setCurrentPage}
          userId={userId}
          isAdminView={isAdminView}
          setReaderCurrentPage={setReaderCurrentPage}
        />

        <div className="flex justify-center items-center py-4">
          <button
            onClick={handleAdminViewClick}
            className={`px-6 py-2 rounded-full font-semibold transition-all duration-300 mx-2 ${
              isAdminView
                ? 'bg-blue-600 text-white shadow-md'
                : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
            }`}
          >
            Admin View
          </button>
          <button
            onClick={() => {
              setIsAdminView(false);
              setReaderCurrentPage('searchBooks'); // Reset to default reader page
            }}
            className={`px-6 py-2 rounded-full font-semibold transition-all duration-300 mx-2 ${
              !isAdminView
                ? 'bg-blue-600 text-white shadow-md'
                : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
            }`}
          >
            Reader View
          </button>
        </div>

        <main className="container mx-auto p-4">
          {isAuthReady ? (
            renderPage()
          ) : (
            <LoadingSpinner />
          )}
        </main>

        <MessageBox message={message} onClose={closeMessage} />

        {/* Edit Book Modal */}
        {editingBook && (
          <EditBook
            bookData={editingBook}
            onClose={() => setEditingBook(null)}
            showMessage={showMessage}
          />
        )}

        {showAdminPasswordModal && (
          <AdminPasswordModal
            onAuthenticate={handleAuthenticateAdmin}
            onCancel={handleCancelAdminAccess}
            showMessage={showMessage}
          />
        )}
      </div>
    </FirebaseContext.Provider>
  );
}



