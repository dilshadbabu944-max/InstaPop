# InstaPop
// Focsta - Minimal Instagram-like Photo App (single-file React)
// How to use:
// 1) Create a new React app (Vite recommended):
//    npm create vite@latest focsta -- --template react
// 2) Replace src/App.jsx with this file's content and ensure Tailwind is set up (or remove Tailwind classes).
// 3) npm install, npm run dev

import React, { useEffect, useState, useRef } from 'react';

export default function App() {
  // Posts stored in localStorage as an array of {id, author, caption, img, likes, comments, createdAt}
  const [posts, setPosts] = useState(() => {
    try {
      return JSON.parse(localStorage.getItem('focsta_posts') || '[]');
    } catch (e) {
      return [];
    }
  });
  const [username, setUsername] = useState(() => localStorage.getItem('focsta_user') || 'Guest');
  const [caption, setCaption] = useState('');
  const [imagePreview, setImagePreview] = useState(null);
  const fileRef = useRef(null);
  const [query, setQuery] = useState('');

  useEffect(() => {
    localStorage.setItem('focsta_posts', JSON.stringify(posts));
  }, [posts]);

  useEffect(() => {
    localStorage.setItem('focsta_user', username);
  }, [username]);

  function handleFile(e) {
    const file = e.target.files?.[0];
    if (!file) return;
    const url = URL.createObjectURL(file);
    setImagePreview({ url, fileName: file.name });
  }

  function createPost() {
    if (!imagePreview) {
      alert('Please add a photo.');
      return;
    }
    const newPost = {
      id: Date.now().toString(),
      author: username || 'Guest',
      caption: caption || '',
      img: imagePreview.url,
      likes: 0,
      likedByMe: false,
      comments: [],
      createdAt: new Date().toISOString(),
    };
    setPosts(prev => [newPost, ...prev]);
    setCaption('');
    setImagePreview(null);
    if (fileRef.current) fileRef.current.value = '';
  }

  function toggleLike(id) {
    setPosts(prev => prev.map(p => p.id === id ? { ...p, likes: p.likedByMe ? p.likes - 1 : p.likes + 1, likedByMe: !p.likedByMe } : p));
  }

  function addComment(id, text) {
    if (!text) return;
    setPosts(prev => prev.map(p => p.id === id ? { ...p, comments: [...p.comments, { id: Date.now().toString(), author: username || 'Guest', text, createdAt: new Date().toISOString() }] } : p));
  }

  function deletePost(id) {
    if (!confirm('Delete this post?')) return;
    setPosts(prev => prev.filter(p => p.id !== id));
  }

  const filtered = posts.filter(p => p.caption.toLowerCase().includes(query.toLowerCase()) || p.author.toLowerCase().includes(query.toLowerCase()));

  return (
    <div className="min-h-screen bg-gray-50 p-4">
      <header className="max-w-3xl mx-auto flex items-center justify-between py-4">
        <h1 className="text-2xl font-bold">Focsta</h1>
        <div className="flex items-center gap-3">
          <input value={username} onChange={e => setUsername(e.target.value)} className="border px-2 py-1 rounded" placeholder="Your name" />
          <input value={query} onChange={e => setQuery(e.target.value)} className="border px-2 py-1 rounded" placeholder="Search posts or authors" />
        </div>
      </header>

      <main className="max-w-3xl mx-auto">
        <section className="bg-white p-4 rounded shadow mb-6">
          <h2 className="font-semibold mb-2">Create Post</h2>
          <div className="flex flex-col gap-2">
            <textarea value={caption} onChange={e => setCaption(e.target.value)} placeholder="Write a caption..." className="border rounded p-2" rows={3} />
            <input ref={fileRef} type="file" accept="image/*" onChange={handleFile} />
            {imagePreview && (
              <div className="mt-2">
                <img src={imagePreview.url} alt="preview" className="max-h-48 object-cover rounded" />
              </div>
            )}
            <div className="flex gap-2">
              <button onClick={createPost} className="px-4 py-2 bg-blue-600 text-white rounded">Post</button>
              <button onClick={() => { setCaption(''); setImagePreview(null); if (fileRef.current) fileRef.current.value=''; }} className="px-4 py-2 border rounded">Clear</button>
            </div>
          </div>
        </section>

        <section>
          {filtered.length === 0 && <p className="text-center text-gray-500">No posts yet — create the first one!</p>}

          {filtered.map(post => (
            <article key={post.id} className="bg-white rounded shadow mb-4 p-3">
              <div className="flex items-center justify-between mb-2">
                <div className="flex items-center gap-3">
                  <div className="w-10 h-10 bg-gray-200 rounded-full flex items-center justify-center text-sm font-semibold">{post.author[0]?.toUpperCase()}</div>
                  <div>
                    <div className="font-semibold">{post.author}</div>
                    <div className="text-xs text-gray-500">{new Date(post.createdAt).toLocaleString()}</div>
                  </div>
                </div>
                <div>
                  <button onClick={() => deletePost(post.id)} className="text-sm text-red-500">Delete</button>
                </div>
              </div>

              <div className="mb-2">
                <img src={post.img} alt="post" className="w-full max-h-96 object-cover rounded" />
              </div>

              <div className="flex items-center gap-4 mb-2">
                <button onClick={() => toggleLike(post.id)} className="font-semibold">{post.likedByMe ? '❤️' : '🤍'} {post.likes}</button>
                <span className="text-sm text-gray-600">{post.comments.length} comments</span>
              </div>

              <div className="mb-2"><span className="font-semibold">{post.author}</span> {post.caption}</div>

              <CommentBox post={post} onAdd={addComment} />

              {post.comments.length > 0 && (
                <div className="mt-2 border-t pt-2">
                  {post.comments.map(c => (
                    <div key={c.id} className="text-sm mb-1">
                      <span className="font-semibold">{c.author}:</span> {c.text}
                    </div>
                  ))}
                </div>
              )}
            </article>
          ))}
        </section>
      </main>

      <footer className="max-w-3xl mx-auto text-center text-xs text-gray-500 mt-6">Built with ❤️ — Focsta (demo). Data saved locally in your browser.</footer>
    </div>
  );
}

function CommentBox({ post, onAdd }) {
  const [text, setText] = useState('');
  return (
    <div className="flex gap-2">
      <input value={text} onChange={e => setText(e.target.value)} placeholder="Add a comment..." className="flex-1 border rounded px-2 py-1" />
      <button onClick={() => { onAdd(post.id, text); setText(''); }} className="px-3 py-1 border rounded">Send</button>
    </div>
  );
}

