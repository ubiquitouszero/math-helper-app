# math-helper-app
Math quiz for grade school math. This is similar to the worksheets sent home with students. 

https://chatgpt.com/share/67981f46-5fa4-8013-a69f-031b3c6f86f6


import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

const gradeLevels = {
  "1st": {
    operations: ["+", "-"],
    skills: {
      addition: "Add single-digit numbers within 10.",
      subtraction: "Subtract single-digit numbers within 10."
    }
  },
  "2nd": {
    operations: ["+", "-"],
    skills: {
      addition: "Fluently add single-digit numbers within 30.",
      subtraction: "Fluently subtract single-digit numbers within 30."
    }
  },
  "3rd": {
    operations: ["+", "-", "*", "/"],
    skills: {
      addition: "Add three-digit numbers.",
      subtraction: "Subtract three-digit numbers.",
      multiplication: "Multiply by numbers up to 10.",
      division: "Divide two-digit numbers by single-digit numbers."
    }
  }
};

const generateQuestions = (operations, gradeLevel) => {
  const max = gradeLevel === "1st" ? 10 : gradeLevel === "2nd" ? 30 : 999;
  const questions = [];

  while (questions.length < 20) {
    const operator = operations[Math.floor(Math.random() * operations.length)];
    let num1 = Math.floor(Math.random() * max) + 1;
    let num2 = Math.floor(Math.random() * max) + 1;

    if (gradeLevel === "2nd" && operator === "-") {
      num1 = Math.max(num1, num2);
      num2 = Math.min(num1, num2);
    }

    if (operator === "-" && num1 < num2) {
      [num1, num2] = [num2, num1];
    }

    if (operator === "/" && num2 === 0) {
      num2 = 1;
    }

    const answer = eval(`${num1} ${operator} ${num2}`);
    questions.push({
      question: `${num1} ${operator} ${num2}`,
      answer: answer.toString(),
    });
  }

  return questions;
};

const MathHelperApp = () => {
  const [questions, setQuestions] = useState([]);
  const [userAnswers, setUserAnswers] = useState(Array(20).fill(""));
  const [timer, setTimer] = useState(60);
  const [gameStarted, setGameStarted] = useState(false);
  const [feedback, setFeedback] = useState("");
  const [report, setReport] = useState(null);
  const [gradeLevel, setGradeLevel] = useState("1st");
  const [operations, setOperations] = useState({
    addition: true,
    subtraction: true,
    multiplication: false,
    division: false,
  });

  const gradeConfig = gradeLevels[gradeLevel];

  useEffect(() => {
    if (gameStarted && timer > 0) {
      const countdown = setInterval(() => setTimer((prev) => prev - 1), 1000);
      return () => clearInterval(countdown);
    } else if (timer === 0) {
      gradeGame();
    }
  }, [gameStarted, timer]);

  const startGame = () => {
    const selectedOperations = Object.keys(operations).filter((op) => operations[op]).map((op) => {
      switch (op) {
        case "addition":
          return "+";
        case "subtraction":
          return "-";
        case "multiplication":
          return "*";
        case "division":
          return "/";
        default:
          return null;
      }
    });

    if (!selectedOperations.length) {
      setFeedback("Please select at least one operation to start the game.");
      return;
    }

    const newQuestions = generateQuestions(selectedOperations, gradeLevel);
    setQuestions(newQuestions);
    setUserAnswers(Array(20).fill(""));
    setGameStarted(true);
    setTimer(60);
    setFeedback("");
    setReport(null);
  };

  const gradeGame = () => {
    const gradedReport = questions.map((q, i) => ({
      ...q,
      userAnswer: userAnswers[i],
      isCorrect: q.answer === userAnswers[i],
    }));

    const correctCount = gradedReport.filter((q) => q.isCorrect).length;
    setReport({ gradedReport, correctCount, total: questions.length });
    setGameStarted(false);
  };

  return (
    <div className="min-h-screen flex flex-col items-center justify-center bg-gray-100 p-4">
      <Card className="max-w-2xl w-full p-4 shadow-lg">
        <CardContent>
          <h1 className="text-xl font-bold text-center mb-4">Math Helper</h1>
          {!gameStarted ? (
            <div className="text-center">
              <p>Select Grade Level:</p>
              <select
                value={gradeLevel}
                onChange={(e) => setGradeLevel(e.target.value)}
                className="p-2 border rounded-md mb-4"
              >
                {Object.keys(gradeLevels).map((grade) => (
                  <option key={grade} value={grade}>{grade}</option>
                ))}
              </select>
              <div>
                <h2 className="text-lg font-bold mb-2">Skills for {gradeLevel}:</h2>
                <ul>
                  {Object.entries(gradeConfig.skills).map(([op, skill]) => (
                    <li key={op}><strong>{op}:</strong> {skill}</li>
                  ))}
                </ul>
              </div>
              <p>Select Operations:</p>
              <div className="flex flex-wrap gap-4 mb-4">
                {Object.keys(operations).map((op) => (
                  <label key={op} className="flex items-center gap-2">
                    <input
                      type="checkbox"
                      checked={operations[op]}
                      onChange={() => setOperations((prev) => ({ ...prev, [op]: !prev[op] }))}
                      className="h-4 w-4"
                    />
                    {op.charAt(0).toUpperCase() + op.slice(1)}
                  </label>
                ))}
              </div>
              <Button onClick={startGame}>Start Game</Button>
              {feedback && <p className="text-red-500 mt-2">{feedback}</p>}
            </div>
          ) : (
            <div>
              <div className={`text-right font-bold mb-2 ${timer <= 10 ? "text-red-500" : "text-black"}`}>Time Left: {timer}s</div>
              {questions.map((q, i) => (
                <div key={i} className="mb-2">
                  <p>{i + 1}. {q.question}</p>
                  <Input
                    type="text"
                    value={userAnswers[i]}
                    onChange={(e) => {
                      const answers = [...userAnswers];
                      answers[i] = e.target.value;
                      setUserAnswers(answers);
                    }}
                  />
                </div>
              ))}
              <Button onClick={gradeGame} className="mt-4">Submit</Button>
            </div>
          )}
          {report && (
            <div>
              <h2 className="text-lg font-bold mb-2">Game Report</h2>
              <p>Questions Attempted: {report.gradedReport.filter(q => q.userAnswer !== "").length}</p>
              <p>Correct: {report.correctCount} / {report.total}</p>
              <ul className="mt-2">
                {report.gradedReport.map((r, i) => (
                  <li key={i} className={r.isCorrect ? "text-green-600" : "text-red-600"}>
                    {r.question} = {r.answer} (You: {r.userAnswer || "N/A"})
                  </li>
                ))}
              </ul>
            </div>
          )}
        </CardContent>
      </Card>
    </div>
  );
};

export default MathHelperApp;
