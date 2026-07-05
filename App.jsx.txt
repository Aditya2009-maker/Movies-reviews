import { useState, useEffect, useRef, useCallback } from "react";
import { Star, Lock, Unlock, X, Upload, Film, Ticket } from "lucide-react";

// --- Supabase connection ---
const SUPABASE_URL = "https://ccmontiitfshlmnowrvy.supabase.co";
const SUPABASE_KEY = "sb_publishable_UEBMs42dAvlA1Talry3xmw_iuqxmNlQ";
const REVIEWS_ENDPOINT = `${SUPABASE_URL}/rest/v1/reviews`;
const SUPABASE_HEADERS = {
  apikey: SUPABASE_KEY,
  Authorization: `Bearer ${SUPABASE_KEY}`,
  "Content-Type": "application/json",
};

const ADMIN_PASSWORD = "director123"; // change this before sharing the link widely

const FONT_LINKS = [
  "https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Libre+Franklin:wght@400;500;600;700&family=Courier+Prime:wght@400;700&display=swap",
];

function useGoogleFonts() {
  useEffect(() => {
    FONT_LINKS.forEach((href) => {
      if (document.querySelector(`link[href="${href}"]`)) return;
      const link = document.createElement("link");
      link.rel = "stylesheet";
      link.href = href;
      document.head.appendChild(link);
    });
  }, []);
}

function resizeImage(file, maxDim = 600, quality = 0.72) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        let { width, height } = img;
        if (width > height && width > maxDim) {
          height = Math.round((height * maxDim) / width);
          width = maxDim;
        } else if (height > maxDim) {
          width = Math.round((width * maxDim) / height);
          height = maxDim;
        }
        const canvas = document.createElement("canvas");
        canvas.width = width;
        canvas.height = height;
        const ctx = canvas.getContext("2d");
        ctx.drawImage(img, 0, 0, width, height);
        resolve(canvas.toDataURL("image/jpeg", quality));
      };
      img.onerror = reject;
      img.src = e.target.result;
    };
    reader.onerror = reject;
    reader.readAsDataURL(file);
  });
}

function StarPicker({ value, onChange, size = 22, readOnly = false }) {
  const [hover, setHover] = useState(0);
  return (
    <div className="flex gap-1">
      {[1, 2, 3, 4, 5].map((n) => {
        const filled = readOnly ? n <= value : n <= (hover || value);
        return (
          <button
            key={n}
            type="button"
            disabled={readOnly}
            onClick={() => onChange && onChange(n)}
            onMouseEnter={() => !readOnly && setHover(n)}
            onMouseLeave={() => !readOnly && setHover(0)}
            className={readOnly ? "cursor-default" : "cursor-pointer transition-transform hover:scale-110"}
            aria-label={`${n} star`}
          >
            <Star
              size={size}
              fill={filled ? "#C9A227" : "none"}
              stroke={filled ? "#C9A227" : "#8a7f6a"}
              strokeWidth={1.5}
              style={filled ? { filter: "drop-shadow(0 0 4px rgba(201,162,39,0.7))" } : {}}
            />
          </button>
        );
      })}
    </div>
  );
}

function TicketStub({ review, index, isAdmin, onDelete }) {
  const dateStr = new Date(review.created_at).toLocaleDateString(undefined, {
    year: "numeric",
    month: "short",
    day: "numeric",
  });
  return (
    <div
      className="relative flex flex-col overflow-hidden"
      style={{
        background: "#F2E8D0",
        borderRadius: 10,
        boxShadow: "0 8px 24px rgba(0,0,0,0.35)",
        fontFamily: "'Libre Franklin', sans-serif",
      }}
    >
      {isAdmin && (
        <button
          onClick={() => onDelete(review.id)}
          className="absolute top-2 right-2 z-10 flex items-center gap-1 px-2 py-1 text-[11px] font-bold tracking-wide"
          style={{
            background: "#7A1F2B",
            color: "#F2E8D0",
            borderRadius: 4,
            transform: "rotate(-4deg)",
            boxShadow: "0 2px 6px rgba(0,0,0,0.4)",
          }}
        >
          <X size={12} /> VOID
        </button>
      )}

      {review.image ? (
        <img
          src={review.image}
          alt={review.title}
          className="w-full object-cover"
          style={{ height: 220 }}
        />
      ) : (
        <div
          className="w-full flex items-center justify-center"
          style={{ height: 220, background: "#241C11" }}
        >
          <Film size={40} color="#8a7f6a" />
        </div>
      )}

      {/* perforated tear divider */}
      <div
        className="relative h-4"
        style={{
          background:
            "repeating-radial-gradient(circle at 0 8px, transparent 0 5px, #F2E8D0 5px 100%)",
          backgroundSize: "16px 16px",
          backgroundPosition: "-8px center",
        }}
      >
        <div
          className="absolute left-0 right-0 top-1/2 border-t border-dashed"
          style={{ borderColor: "#a8977a" }}
        />
      </div>

      <div className="p-4 pt-3 flex flex-col gap-2">
        <h3
          className="leading-tight"
          style={{
            fontFamily: "'Bebas Neue', sans-serif",
            fontSize: 26,
            letterSpacing: 0.5,
            color: "#241C11",
          }}
        >
          {review.title}
        </h3>

        <StarPicker value={review.rating} readOnly size={17} />

        <p
          style={{
            fontFamily: "'Courier Prime', monospace",
            fontSize: 13.5,
            lineHeight: 1.5,
            color: "#3a3126",
            whiteSpace: "pre-wrap",
          }}
        >
          {review.review_text}
        </p>

        <div
          className="flex items-center justify-between pt-2 mt-1 border-t"
          style={{ borderColor: "#d9cba9", fontSize: 11, color: "#8a7f6a" }}
        >
          <span>— {review.name || "Anonymous"}</span>
          <span>{dateStr}</span>
        </div>

        <div
          className="flex items-center justify-between mt-1 pt-2 border-t border-dashed"
          style={{ borderColor: "#c7b78f" }}
        >
          <span
            style={{
              fontFamily: "'Courier Prime', monospace",
              fontSize: 10,
              letterSpacing: 2,
              color: "#a8977a",
            }}
          >
            ADMIT ONE
          </span>
          <span
            style={{
              fontFamily: "'Courier Prime', monospace",
              fontSize: 10,
              color: "#a8977a",
            }}
          >
            NO. {String(index + 1).padStart(4, "0")}
          </span>
        </div>
      </div>
    </div>
  );
}

export default function TheBalcony() {
  useGoogleFonts();

  const [reviews, setReviews] = useState([]);
  const [loading, setLoading] = useState(true);
  const [loadError, setLoadError] = useState(false);

  const [title, setTitle] = useState("");
  const [name, setName] = useState("");
  const [rating, setRatingVal] = useState(0);
  const [text, setText] = useState("");
  const [imageData, setImageData] = useState(null);
  const [imageProcessing, setImageProcessing] = useState(false);
  const [submitting, setSubmitting] = useState(false);
  const [formError, setFormError] = useState("");
  const fileInputRef = useRef(null);

  const [isAdmin, setIsAdmin] = useState(false);
  const [showLogin, setShowLogin] = useState(false);
  const [pwInput, setPwInput] = useState("");
  const [pwError, setPwError] = useState("");

  const loadReviews = useCallback(async () => {
    try {
      const res = await fetch(
        `${REVIEWS_ENDPOINT}?select=*&order=created_at.desc`,
        { headers: SUPABASE_HEADERS }
      );
      if (!res.ok) throw new Error("Failed to load reviews");
      const data = await res.json();
      setReviews(Array.isArray(data) ? data : []);
      setLoadError(false);
    } catch (err) {
      setLoadError(true);
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    loadReviews();
  }, [loadReviews]);

  const handleFile = async (e) => {
    const file = e.target.files?.[0];
    if (!file) return;
    if (!file.type.startsWith("image/")) {
      setFormError("Please choose an image file.");
      return;
    }
    setFormError("");
    setImageProcessing(true);
    try {
      const dataUrl = await resizeImage(file);
      setImageData(dataUrl);
    } catch {
      setFormError("Couldn't process that image. Try another file.");
    } finally {
      setImageProcessing(false);
    }
  };

  const resetForm = () => {
    setTitle("");
    setName("");
    setRatingVal(0);
    setText("");
    setImageData(null);
    if (fileInputRef.current) fileInputRef.current.value = "";
  };

  const submitReview = async (e) => {
    e.preventDefault();
    if (!title.trim()) return setFormError("The movie needs a title.");
    if (!rating) return setFormError("Pick a star rating.");
    if (!text.trim()) return setFormError("Write a few words about the movie.");

    setSubmitting(true);
    setFormError("");
    const newReview = {
      id: `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`,
      title: title.trim(),
      name: name.trim(),
      rating,
      review_text: text.trim(),
      image: imageData,
    };
    try {
      const res = await fetch(REVIEWS_ENDPOINT, {
        method: "POST",
        headers: { ...SUPABASE_HEADERS, Prefer: "return=representation" },
        body: JSON.stringify(newReview),
      });
      if (!res.ok) throw new Error("Failed to save review");
      const [saved] = await res.json();
      setReviews((prev) => [saved, ...prev]);
      resetForm();
    } catch (err) {
      setFormError("Couldn't post your review — check your connection and try again.");
    } finally {
      setSubmitting(false);
    }
  };

  const handleDelete = async (id) => {
    const prev = reviews;
    setReviews(reviews.filter((r) => r.id !== id));
    try {
      const res = await fetch(`${REVIEWS_ENDPOINT}?id=eq.${encodeURIComponent(id)}`, {
        method: "DELETE",
        headers: SUPABASE_HEADERS,
      });
      if (!res.ok) throw new Error("Failed to delete");
    } catch (err) {
      setReviews(prev);
      setLoadError(true);
    }
  };

  const attemptLogin = (e) => {
    e.preventDefault();
    if (pwInput === ADMIN_PASSWORD) {
      setIsAdmin(true);
      setShowLogin(false);
      setPwInput("");
      setPwError("");
    } else {
      setPwError("Wrong password.");
    }
  };

  return (
    <div
      className="min-h-screen w-full"
      style={{ background: "#171310", fontFamily: "'Libre Franklin', sans-serif" }}
    >
      {/* Marquee header */}
      <div
        className="relative py-10 px-6 text-center border-b-4"
        style={{
          borderColor: "#C9A227",
          background:
            "radial-gradient(ellipse at top, rgba(122,31,43,0.35), transparent 70%), #1c1712",
        }}
      >
        <button
          onClick={() => (isAdmin ? setIsAdmin(false) : setShowLogin(true))}
          className="absolute top-4 right-4 flex items-center gap-1.5 px-3 py-1.5 text-xs font-semibold"
          style={{
            color: isAdmin ? "#171310" : "#C9A227",
            background: isAdmin ? "#C9A227" : "transparent",
            border: "1px solid #C9A227",
            borderRadius: 999,
          }}
        >
          {isAdmin ? <Unlock size={13} /> : <Lock size={13} />}
          {isAdmin ? "Admin" : "Admin login"}
        </button>

        <div className="flex items-center justify-center gap-3">
          <Ticket size={28} color="#C9A227" />
          <h1
            style={{
              fontFamily: "'Bebas Neue', sans-serif",
              fontSize: "clamp(36px, 6vw, 64px)",
              color: "#F2E8D0",
              letterSpacing: 4,
              textShadow: "0 0 18px rgba(201,162,39,0.35)",
            }}
          >
            THE BALCONY
          </h1>
          <Ticket size={28} color="#C9A227" style={{ transform: "scaleX(-1)" }} />
        </div>
        <p
          style={{
            fontFamily: "'Courier Prime', monospace",
            color: "#a8977a",
            fontSize: 13,
            letterSpacing: 2,
            marginTop: 6,
          }}
        >
          REVIEWS FROM THE CHEAP SEATS · ANYONE CAN POST
        </p>
      </div>

      {showLogin && !isAdmin && (
        <div
          className="fixed inset-0 z-50 flex items-center justify-center px-4"
          style={{ background: "rgba(0,0,0,0.6)" }}
          onClick={() => setShowLogin(false)}
        >
          <form
            onClick={(e) => e.stopPropagation()}
            onSubmit={attemptLogin}
            className="w-full max-w-xs p-5 flex flex-col gap-3"
            style={{ background: "#F2E8D0", borderRadius: 10 }}
          >
            <h2
              style={{ fontFamily: "'Bebas Neue', sans-serif", fontSize: 22, color: "#241C11" }}
            >
              Admin login
            </h2>
            <input
              type="password"
              autoFocus
              value={pwInput}
              onChange={(e) => setPwInput(e.target.value)}
              placeholder="Password"
              className="px-3 py-2 outline-none"
              style={{ border: "1px solid #a8977a", borderRadius: 6, background: "#fff" }}
            />
            {pwError && <p style={{ color: "#7A1F2B", fontSize: 12 }}>{pwError}</p>}
            <div className="flex gap-2">
              <button
                type="submit"
                className="flex-1 py-2 font-semibold text-sm"
                style={{ background: "#7A1F2B", color: "#F2E8D0", borderRadius: 6 }}
              >
                Unlock
              </button>
              <button
                type="button"
                onClick={() => setShowLogin(false)}
                className="flex-1 py-2 font-semibold text-sm"
                style={{ background: "#e4d6b3", color: "#241C11", borderRadius: 6 }}
              >
                Cancel
              </button>
            </div>
          </form>
        </div>
      )}

      <div className="max-w-5xl mx-auto px-4 md:px-6 py-10 flex flex-col gap-10">
        {/* Submission form - ticket booth */}
        <form
          onSubmit={submitReview}
          className="p-5 md:p-6 flex flex-col gap-4"
          style={{
            background: "#F2E8D0",
            borderRadius: 10,
            boxShadow: "0 8px 24px rgba(0,0,0,0.35)",
          }}
        >
          <h2
            style={{
              fontFamily: "'Bebas Neue', sans-serif",
              fontSize: 24,
              letterSpacing: 1,
              color: "#241C11",
            }}
          >
            Post a review
          </h2>

          <div className="grid md:grid-cols-2 gap-4">
            <div className="flex flex-col gap-3">
              <input
                value={title}
                onChange={(e) => setTitle(e.target.value)}
                placeholder="Movie title"
                className="px-3 py-2 outline-none text-sm"
                style={{ border: "1px solid #c7b78f", borderRadius: 6, background: "#fff" }}
              />
              <input
                value={name}
                onChange={(e) => setName(e.target.value)}
                placeholder="Your name (optional)"
                className="px-3 py-2 outline-none text-sm"
                style={{ border: "1px solid #c7b78f", borderRadius: 6, background: "#fff" }}
              />
              <div className="flex items-center gap-3">
                <span style={{ fontSize: 13, color: "#241C11" }}>Rating:</span>
                <StarPicker value={rating} onChange={setRatingVal} />
              </div>
              <label
                className="flex items-center justify-center gap-2 py-2 text-sm cursor-pointer font-medium"
                style={{
                  border: "1px dashed #a8977a",
                  borderRadius: 6,
                  color: "#5a4f3d",
                  background: imageData ? "#e9dfc4" : "transparent",
                }}
              >
                <Upload size={14} />
                {imageProcessing ? "Processing..." : imageData ? "Poster added — change" : "Upload a movie poster/pic"}
                <input
                  ref={fileInputRef}
                  type="file"
                  accept="image/*"
                  onChange={handleFile}
                  className="hidden"
                />
              </label>
              {imageData && (
                <img
                  src={imageData}
                  alt="preview"
                  className="rounded"
                  style={{ height: 100, objectFit: "cover" }}
                />
              )}
            </div>

            <textarea
              value={text}
              onChange={(e) => setText(e.target.value)}
              placeholder="What did you think of it?"
              className="px-3 py-2 outline-none text-sm resize-none"
              style={{
                border: "1px solid #c7b78f",
                borderRadius: 6,
                background: "#fff",
                minHeight: 140,
                fontFamily: "'Courier Prime', monospace",
              }}
            />
          </div>

          {formError && <p style={{ color: "#7A1F2B", fontSize: 13 }}>{formError}</p>}

          <button
            type="submit"
            disabled={submitting || imageProcessing}
            className="self-start px-5 py-2.5 font-semibold text-sm tracking-wide disabled:opacity-60"
            style={{ background: "#7A1F2B", color: "#F2E8D0", borderRadius: 6 }}
          >
            {submitting ? "Posting..." : "Post review"}
          </button>
        </form>

        {/* Reviews grid */}
        {loading ? (
          <p style={{ color: "#a8977a", textAlign: "center" }}>Loading reviews…</p>
        ) : reviews.length === 0 ? (
          <p style={{ color: "#a8977a", textAlign: "center" }}>
            No reviews yet — be the first to post one.
          </p>
        ) : (
          <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-6">
            {reviews.map((r, i) => (
              <TicketStub
                key={r.id}
                review={r}
                index={reviews.length - 1 - i}
                isAdmin={isAdmin}
                onDelete={handleDelete}
              />
            ))}
          </div>
        )}
        {loadError && (
          <p style={{ color: "#7A1F2B", textAlign: "center", fontSize: 12 }}>
            Couldn't reach the database — check your connection and try again.
          </p>
        )}
      </div>
    </div>
  );
}
