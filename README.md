# simple-interest.khk
simple interest calculator.
import React, { useState, useEffect } from "react";
import { motion } from "framer-motion";

// Default export: single-file React component (Tailwind classes used)
export default function SimpleInterestCalculator() {
  const [principal, setPrincipal] = useState(1000);
  const [rate, setRate] = useState(5);
  const [time, setTime] = useState(1);
  const [interest, setInterest] = useState(0);
  const [amount, setAmount] = useState(0);
  const [history, setHistory] = useState([]);
  const [error, setError] = useState("");

  useEffect(() => {
    // load history from localStorage
    try {
      const raw = localStorage.getItem("si_history_v1");
      if (raw) setHistory(JSON.parse(raw));
    } catch (e) {
      // ignore
    }
  }, []);

  useEffect(() => {
    calculate();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [principal, rate, time]);

  function validateInputs(p, r, t) {
    if (isNaN(p) || isNaN(r) || isNaN(t)) return "Please enter valid numeric values.";
    if (p <= 0) return "Principal must be greater than zero.";
    if (r < 0) return "Rate cannot be negative.";
    if (t <= 0) return "Time must be greater than zero.";
    return "";
  }

  function calculate() {
    const p = Number(principal);
    const r = Number(rate);
    const t = Number(time);

    const vErr = validateInputs(p, r, t);
    setError(vErr);
    if (vErr) {
      setInterest(0);
      setAmount(0);
      return;
    }

    const si = (p * r * t) / 100;
    const tot = p + si;
    setInterest(Number(si.toFixed(2)));
    setAmount(Number(tot.toFixed(2)));
  }

  function saveToHistory() {
    if (error) return;
    const entry = {
      id: Date.now(),
      principal: Number(principal),
      rate: Number(rate),
      time: Number(time),
      interest,
      amount,
      date: new Date().toISOString(),
    };
    const next = [entry, ...history].slice(0, 20);
    setHistory(next);
    try {
      localStorage.setItem("si_history_v1", JSON.stringify(next));
    } catch (e) {}
  }

  function resetAll() {
    setPrincipal(1000);
    setRate(5);
    setTime(1);
    setError("");
  }

  function copyResult() {
    const text = `Principal: ${principal}\nRate: ${rate}%\nTime: ${time}\nSimple Interest: ${interest}\nTotal Amount: ${amount}`;
    navigator.clipboard?.writeText(text);
  }

  function downloadCSV() {
    if (!history.length) return;
    const rows = [
      ["date", "principal", "rate", "time", "interest", "amount"],
      ...history.map((h) => [h.date, h.principal, h.rate, h.time, h.interest, h.amount]),
    ];
    const csv = rows.map((r) => r.join(",")).join("\n");
    const blob = new Blob([csv], { type: "text/csv" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "simple_interest_history.csv";
    a.click();
    URL.revokeObjectURL(url);
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-b from-white to-gray-50 p-6">
      <motion.div
        initial={{ opacity: 0, scale: 0.98 }}
        animate={{ opacity: 1, scale: 1 }}
        transition={{ duration: 0.35 }}
        className="w-full max-w-3xl bg-white rounded-2xl shadow-lg p-6 md:p-10"
      >
        <h1 className="text-2xl md:text-3xl font-semibold mb-2">Simple Interest Calculator</h1>
        <p className="text-sm text-gray-500 mb-6">Quickly compute simple interest and total amount. Inputs accept decimals.</p>

        <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
          <label className="flex flex-col">
            <span className="text-sm font-medium mb-1">Principal (₹)</span>
            <input
              type="number"
              step="0.01"
              min="0"
              value={principal}
              onChange={(e) => setPrincipal(e.target.value)}
              className="px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-200"
              aria-label="Principal amount"
            />
          </label>

          <label className="flex flex-col">
            <span className="text-sm font-medium mb-1">Rate (% per year)</span>
            <input
              type="number"
              step="0.01"
              value={rate}
              onChange={(e) => setRate(e.target.value)}
              className="px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-200"
              aria-label="Interest rate"
            />
          </label>

          <label className="flex flex-col">
            <span className="text-sm font-medium mb-1">Time (years)</span>
            <input
              type="number"
              step="0.01"
              min="0"
              value={time}
              onChange={(e) => setTime(e.target.value)}
              className="px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-200"
              aria-label="Time in years"
            />
          </label>
        </div>

        {error ? (
          <div className="text-sm text-red-600 mb-4">{error}</div>
        ) : (
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
            <div className="p-4 rounded-lg border">
              <div className="text-sm text-gray-500">Simple Interest</div>
              <div className="text-2xl font-semibold mt-1">₹ {interest}</div>
            </div>

            <div className="p-4 rounded-lg border">
              <div className="text-sm text-gray-500">Total Amount (Principal + Interest)</div>
              <div className="text-2xl font-semibold mt-1">₹ {amount}</div>
            </div>
          </div>
        )}

        <div className="flex gap-3 mb-6">
          <button
            onClick={saveToHistory}
            className="px-4 py-2 rounded-lg bg-indigo-600 text-white font-medium shadow-sm hover:bg-indigo-700"
            aria-label="Save to history"
          >
            Save
          </button>

          <button
            onClick={copyResult}
            className="px-4 py-2 rounded-lg border font-medium hover:bg-gray-50"
            aria-label="Copy result"
          >
            Copy
          </button>

          <button
            onClick={resetAll}
            className="px-4 py-2 rounded-lg border font-medium hover:bg-gray-50"
            aria-label="Reset inputs"
          >
            Reset
          </button>

          <button
            onClick={downloadCSV}
            className="ml-auto px-4 py-2 rounded-lg border font-medium hover:bg-gray-50"
            aria-label="Download CSV"
          >
            Download History
          </button>
        </div>

        <div>
          <h2 className="text-lg font-medium mb-2">Recent calculations</h2>
          {history.length === 0 ? (
            <div className="text-sm text-gray-500">No saved calculations yet. Click "Save" to store results locally.</div>
          ) : (
            <div className="space-y-2 max-h-48 overflow-auto">
              {history.map((h) => (
                <div key={h.id} className="p-3 rounded-lg border flex items-center justify-between">
                  <div>
                    <div className="text-sm">₹{h.principal} @ {h.rate}% for {h.time}y</div>
                    <div className="text-xs text-gray-500">SI: ₹{h.interest} • Total: ₹{h.amount} • {new Date(h.date).toLocaleString()}</div>
                  </div>
                  <div>
                    <button
                      onClick={() => {
                        // remove this entry
                        const next = history.filter((x) => x.id !== h.id);
                        setHistory(next);
                        localStorage.setItem("si_history_v1", JSON.stringify(next));
                      }}
                      className="px-3 py-1 text-sm rounded bg-red-50 text-red-600 border"
                      aria-label="Delete history entry"
                    >
                      Delete
                    </button>
                  </div>
                </div>
              ))}
            </div>
          )}
        </div>

        <footer className="mt-6 text-xs text-gray-400">
          Simple Interest = Principal × Rate × Time / 100 • Values rounded to 2 decimals.
        </footer>
      </motion.div>
    </div>
  );
}
