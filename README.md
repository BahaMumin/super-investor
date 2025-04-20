import { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Slider } from "@/components/ui/slider";
import { Progress } from "@/components/ui/progress";

const assets = [
  { name: "Акции", volatility: 0.1, expectedReturn: 0.08 },
  { name: "Облигации", volatility: 0.02, expectedReturn: 0.04 },
  { name: "Криптовалюта", volatility: 0.2, expectedReturn: 0.15 },
  { name: "Недвижимость", volatility: 0.05, expectedReturn: 0.06 },
];

export default function SuperInvestor() {
  const [capital, setCapital] = useState(10000);
  const [portfolio, setPortfolio] = useState([25, 25, 25, 25]);
  const [day, setDay] = useState(0);
  const [history, setHistory] = useState([10000]);
  const [tip, setTip] = useState("");

  useEffect(() => {
    if (day > 0) {
      let newCapital = 0;
      assets.forEach((asset, i) => {
        const dailyChange = asset.expectedReturn / 250 + (Math.random() - 0.5) * asset.volatility;
        const portion = portfolio[i] / 100;
        newCapital += capital * portion * (1 + dailyChange);
      });
      // Add unallocated capital as cash
      const allocatedPercentage = portfolio.reduce((a, b) => a + b, 0);
      newCapital += capital * (1 - allocatedPercentage / 100);
      setCapital(newCapital);
      setHistory(prev => [...prev, newCapital]);

      if (day % 5 === 0) {
        setTip(Math.random() > 0.5 ? "Диверсифицируй портфель!" : "Не поддавайся панике!");
      }
    }
  }, [day]);

  const nextDay = () => setDay(day + 1);

  const resetGame = () => {
    setCapital(10000);
    setPortfolio([25, 25, 25, 25]);
    setDay(0);
    setHistory([10000]);
    setTip("");
  };

  const totalAllocation = portfolio.reduce((a, b) => a + b, 0);

  return (
    <div className="p-4 space-y-4">
      <h1 className="text-2xl font-bold">Супер Инвестор</h1>
      <p>Цель: Увеличь капитал, минимизируя риски за 30 дней!</p>
      {day >= 30 && (
        <div className="bg-green-100 p-4 rounded-md">
          <p className="text-green-700 font-semibold">Игра завершена! Ваш итоговый капитал: {capital.toFixed(2)} ₽</p>
        </div>
      )}
      <Card>
        <CardContent className="space-y-2">
          <p>Капитал: {capital.toFixed(2)} ₽</p>
          <Progress value={(day / 30) * 100} />
          <p>День {day} из 30</p>
          {assets.map((asset, i) => (
            <div key={i} className="space-y-1">
              <label>{asset.name}: {portfolio[i]}%</label>
              <Slider
                value={[portfolio[i]]}
                max={100}
                step={1}
                onValueChange={(val) => {
                  const newPortfolio = [...portfolio];
                  const newValue = val[0];
                  const currentSum = portfolio.reduce((a, b) => a + b, 0) - portfolio[i] + newValue;
                  if (currentSum <= 100) {
                    newPortfolio[i] = newValue;
                    setPortfolio(newPortfolio);
                  }
                }}
              />
            </div>
          ))}
          <p className={totalAllocation < 100 ? "text-orange-600" : "text-gray-600"}>
            Распределено: {totalAllocation}% {totalAllocation < 100 && "(Остаток в кэше)"}
          </p>
          <p className="text-blue-600 font-semibold">Подсказка: {tip}</p>
          <div className="flex space-x-2">
            <Button onClick={nextDay} disabled={day >= 30}>Следующий день</Button>
            <Button onClick={resetGame} variant="outline">Сброс</Button>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}
