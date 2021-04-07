    def devastorSimulationEra(self, pair):
        # вывод длины массива свечей по паре
        print('свечей всего:', len(self.devastorCandlesArray))
        # обнуляем массив стоп-лоссов
        self.devastorLossArray = []
        # обнуляем массив байлимов
        self.devastorBuyLimitArray = []
        # динамический стоп-лосс
        self.devastorDynamicLoss = -1
        # обнуялем массив лучших EMA
        self.devastorBestEMAArray = []
        # обнуляем массив парметров
        self.devastorParameterArray = []
        # закидываем в соответствующие переменные curr1 и curr2
        self.devastorCurr1 = pair[0:pair.index('_')]
        self.devastorCurr2 = pair[pair.index('_') + 1:len(pair)]
        # выводим текущую пару
        print('Имитация торгов по паре', self.devastorCurrentPair)
        # обнуляем переменную ячейки
        self.devastorCellCurrent = 0
        # словарь статистики
        self.devastorAverages = {"Profit": [], "Balance": [], "Trades": [], "Percent": []}
        # для каждого профита из сиска профитов
        for profit in PROFITS:
            # минимальный профит от каждой сделки
            self.devastorProfitPercent = float(profit / 100)
            # счетчики стоп-лоссов по тренду и против
            self.devastorLossesTrend = 0
            self.devastorLossesAntiTrend = 0
            # для каждого стоп-лосса из массива стоп-лоссов
            startLOSS = 0
            finishLOSS = LOSS_PERIOD
            if LOSS_FIX:
                startLOSS = LOSS_PERIOD
                finishLOSS = LOSS_PERIOD + 1
            for loss in range(startLOSS, finishLOSS, LOSS_STEP):
                self.devastorLoss = loss
                # для каждого EMA
                startEMA = 0
                finishEMA = EMA_PERIOD
                if EMA_FIX:
                    startEMA = EMA_PERIOD
                    finishEMA = EMA_PERIOD + 1
                for currentEMA in range(startEMA, finishEMA):
                    self.devastorEMACurrent = currentEMA

                    startBUY_LIMIT = 1
                    finishBUY_LIMIT = BUY_TRIGGER_LIMIT
                    if BUY_LIMIT_FIX:
                        startBUY_LIMIT = BUY_TRIGGER_LIMIT
                        finishBUY_LIMIT = BUY_TRIGGER_LIMIT + 1
                    for bl in range(startBUY_LIMIT, finishBUY_LIMIT, BUY_LIMIT_STEP):
                        startPARAMETER = PARAMETER_START
                        finishPARAMETER = PARAMETER
                        if PARAMETER_FIX:
                            startPARAMETER = PARAMETER
                            finishPARAMETER = PARAMETER + 1
                        for parameter in range(startPARAMETER, finishPARAMETER, PARAMETER_STEP):
                            # здесь устанавливаем какая из переменных будет изменяемым параметром
                            # needed_var = parameter

                            self.devastorBuyLimit = bl
                            self.devastorStartBalance = START_BALANCE
                            # запускаем один цикл симуляции торгов
                            self.devastorTradeSimulation()
                            # кладем полученные профит, баланс и количество сделок в соответствующие словари
                            self.devastorAverages["Profit"].append(self.devastorBalance[self.devastorCurr2] - self.devastorStartBalance)
                            self.devastorAverages["Balance"].append(self.devastorBalance[self.devastorCurr2])
                            self.devastorAverages["Trades"].append(self.devastorDeals)
                            # кладем в массив стоп-лоссов текущий стоп-лосс
                            self.devastorLossArray.append(self.devastorLoss/10)
                            # кладем в массив бай-лимитов лучший лимит
                            self.devastorBuyLimitArray.append(self.devastorBuyLimit)
                            # кладем в массив EMA текущий EMA
                            self.devastorBestEMAArray.append(currentEMA)

                            # кладем в массив параметров лучший параметр
                            self.devastorParameterArray.append(parameter)

                            # кладем текущий процент прибыли со сделки в соответствующий словарь
                            self.devastorAverages['Percent'].append(profit)
                            # инкрементируем переменную ячейки
                            self.devastorCellCurrent += 1


        self.devastorBestIndex = self.devastorAverages["Balance"].index(max(self.devastorAverages["Balance"]))

        print('Максимальная просадка:', min(self.devastorAverages["Balance"]) - START_BALANCE)

        # вывод информации
        print('Лучший индес:', self.devastorBestIndex)
        print('Лучшие показатели для пары ' + self.devastorCurrentPair + ':',
              'прибыль:', float(self.devastorAverages['Balance'][self.devastorBestIndex]) - START_BALANCE,
              'баланс:', self.devastorAverages['Balance'][self.devastorBestIndex],
              'сделки:', self.devastorAverages['Trades'][self.devastorBestIndex],
              '|| лучшее EMA:', self.devastorBestEMAArray[self.devastorBestIndex],
              'стоп-лосс:', str(self.devastorLossArray[self.devastorBestIndex]) + '%',
              'байлим:', str(self.devastorBuyLimitArray[self.devastorBestIndex]),
              'максимальный байлим за все время:', self.devastorLastMaxBuyLim,
              'лучший PARAMETER =', self.devastorParameterArray[self.devastorBestIndex])
        self.devastorProfit = 100 * (float(self.devastorAverages['Balance'][self.devastorBestIndex]) - float(self.devastorStartBalance)) / float(self.devastorStartBalance)
        self.devastorSUMM += float(self.devastorAverages['Balance'][self.devastorBestIndex]) - float(self.devastorStartBalance)
        self.devastorOptimumLoss = self.devastorLossArray[self.devastorBestIndex]

        # записываем данные в csv-файл
        if CSV_WRITE:
            with open(self.devastorDirName + MEMORY_FOLDER + 'devastorPairs_' + TIME_PERIOD  + '.csv', "a", newline="") as file:
                writer = csv.writer(file)
                # пишем в csv данные:
                writer.writerow(['пара','баланс','выручка','лосс','байлим','сделки','период'])
                writer.writerow([
                    self.devastorCurrentPair,
                    self.devastorAverages['Balance'][self.devastorBestIndex],
                    float(self.devastorAverages['Balance'][self.devastorBestIndex]) - START_BALANCE,
                    str(self.devastorLossArray[self.devastorBestIndex]) + '%',
                    str(self.devastorBuyLimitArray[self.devastorBestIndex]),
                    self.devastorAverages['Trades'][self.devastorBestIndex],
                    len(self.devastorY)
                ])
