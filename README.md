import React, { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import axios from "axios";

export default function Home() {
  const [formData, setFormData] = useState({ name: "", email: "", message: "" });
  const [loading, setLoading] = useState(false);
  const [successMessage, setSuccessMessage] = useState("");
  const [errorMessage, setErrorMessage] = useState("");
  const [submissions, setSubmissions] = useState([]);
  const [search, setSearch] = useState("");
  const [sortOrder, setSortOrder] = useState("asc");

  useEffect(() => {
    fetchSubmissions();
  }, []);

  const fetchSubmissions = async () => {
    try {
      console.log("Fetching submissions...");
      const response = await axios.get("http://localhost:5000/api/submissions", {
        headers: {
          "Content-Type": "application/json",
        },
      });
      setSubmissions(response.data);
    } catch (error) {
      console.error("Error fetching submissions:", error);
      setErrorMessage("Failed to fetch submissions. Please try again later.");
    }
  };

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setSuccessMessage("");
    setErrorMessage("");
    try {
      console.log("Submitting form with data:", formData);
      await axios.post("http://localhost:5000/api/contact", formData, {
        headers: {
          "Content-Type": "application/json",
        },
      });
      setSuccessMessage("Form submitted successfully!");
      setFormData({ name: "", email: "", message: "" });
      fetchSubmissions();
    } catch (error) {
      console.error("Error submitting form:", error);
      setErrorMessage("Error submitting form. Please try again.");
    }
    setLoading(false);
  };

  const filteredSubmissions = submissions.filter((submission) =>
    submission.name.toLowerCase().includes(search.toLowerCase()) ||
    submission.email.toLowerCase().includes(search.toLowerCase()) ||
    submission.message.toLowerCase().includes(search.toLowerCase())
  );

  const sortedSubmissions = [...filteredSubmissions].sort((a, b) => {
    if (sortOrder === "asc") {
      return new Date(a.submittedAt) - new Date(b.submittedAt);
    } else {
      return new Date(b.submittedAt) - new Date(a.submittedAt);
    }
  });

  return (
    <div className="min-h-screen bg-gray-100 flex flex-col items-center p-6">
      <h1 className="text-3xl font-bold mb-4">Welcome to Track.ng</h1>
      <p className="text-gray-700 mb-6">Data collection and analysis for Oil & Gas, Maritime, and Supply Chain industries.</p>
      
      <Card className="w-full max-w-md p-4 bg-white shadow-lg rounded-2xl mb-6">
        <CardContent>
          <h2 className="text-xl font-semibold mb-4">Contact Us</h2>
          {successMessage && <p className="text-green-600">{successMessage}</p>}
          {errorMessage && <p className="text-red-600">{errorMessage}</p>}
          <form onSubmit={handleSubmit} className="space-y-4">
            <Input 
              type="text" 
              name="name" 
              placeholder="Your Name" 
              value={formData.name} 
              onChange={handleChange} 
              required 
            />
            <Input 
              type="email" 
              name="email" 
              placeholder="Your Email" 
              value={formData.email} 
              onChange={handleChange} 
              required 
            />
            <Textarea 
              name="message" 
              placeholder="Your Message" 
              value={formData.message} 
              onChange={handleChange} 
              required 
            />
            <Button type="submit" className="w-full" disabled={loading}>
              {loading ? "Submitting..." : "Submit"}
            </Button>
          </form>
        </CardContent>
      </Card>
      
      <Card className="w-full max-w-2xl p-4 bg-white shadow-lg rounded-2xl">
        <CardContent>
          <h2 className="text-xl font-semibold mb-4">Submissions Dashboard</h2>
          <Input 
            type="text" 
            placeholder="Search submissions..." 
            value={search} 
            onChange={(e) => setSearch(e.target.value)} 
            className="mb-4"
          />
          <Button onClick={() => setSortOrder(sortOrder === "asc" ? "desc" : "asc")} className="mb-4">
            Sort by Date ({sortOrder === "asc" ? "Oldest First" : "Newest First"})
          </Button>
          {sortedSubmissions.length === 0 ? (
            <p>No submissions found.</p>
          ) : (
            <ul className="space-y-2">
              {sortedSubmissions.map((submission, index) => (
                <li key={index} className="border-b pb-2 mb-2">
                  <p><strong>Name:</strong> {submission.name}</p>
                  <p><strong>Email:</strong> {submission.email}</p>
                  <p><strong>Message:</strong> {submission.message}</p>
                  <p className="text-gray-500 text-sm">{new Date(submission.submittedAt).toLocaleString()}</p>
                </li>
              ))}
            </ul>
          )}
        </CardContent>
      </Card>
    </div>
  );
}
