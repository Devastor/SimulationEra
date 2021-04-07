    def devastorMainLoop(self):
        self.devastorNowMinute = dt.datetime.now().time().minute
        if (self.devastorNowMinute != self.devastorLastMinute):
            self.devastorLastMinute = self.devastorNowMinute

            # вычисляем абсциссу
            self.abscissStr = str(dt.datetime.now().hour) + \
                                       ':' + \
                                       str(dt.datetime.now().minute)
            # взываем к API чтобы достать сигналы
            self.devastorTradeArray = self.devastorCallAPI('trades', pair=self.devastorCurrentPair)[self.devastorCurrentPair]
            self.devastorTickerArray = self.devastorCallAPI('ticker')[self.devastorCurrentPair]
            # сигналы
            self.devastorLastBuyArr.append(float(self.devastorTickerArray['buy_price']))        # последний ордер на покупку
            self.devastorLastSellArr.append(float(self.devastorTickerArray['sell_price']))      # последний ордер на продажу
            self.devastorLastHighArr.append(float(self.devastorTickerArray['high']))            # хай последней свечи
            self.devastorLastLowArr.append(float(self.devastorTickerArray['low']))              # лоу последней свечи
            self.devastorLastAvgArr.append(float(self.devastorTickerArray['avg']))              # мид последней свечи
            self.devastorLastVolArr.append(float(self.devastorTickerArray['vol']))              # валюта в минуту
            self.devastorLastVolCurrArr.append(float(self.devastorTickerArray['vol_curr']))     # объем торгов

            # цена последней сделки
            self.devastorLastTrade = float(self.devastorTradeArray[0]['price'])

            # выводим текущую цену, прогноз, качество прогноза
            try:
                # вычисляем реальный тренд и пресказанный
                trendSign = math.copysign(1,
                    self.devastorPricePredictArr[len(self.devastorPricePredictArr) - 1 - self.PREDICT_LIMIT] - self.devastorLastTrade)
                trendPredictSign = math.copysign(1,
                    self.devastorPredictFutureArray[len(self.devastorPredictFutureArray) - 1 - self.PREDICT_LIMIT] - self.devastorLastTrade)

                if trendSign > 0:
                    self.devastorRealTrend = 'DOWN'
                else:
                    self.devastorRealTrend = 'UP'

                if trendPredictSign > 0:
                    self.devastorPredictedTrend = 'DOWN'
                else:
                    self.devastorPredictedTrend = 'UP'

                if self.devastorPredictedTrend == self.devastorRealTrend:
                    self.trendPredicted = True
                    self.devastorTrendQualityPredictionArray.append(1)
                else:
                    self.trendPredicted = False
                    self.devastorTrendQualityPredictionArray.append(0)
            except Exception as e:
                print("ERROR:", e)

            try:
                self.trendPredictionPercent = ((sum(self.devastorTrendQualityPredictionArray) / len(self.devastorTrendQualityPredictionArray))) * 100
                # считаем процент расхождения предсказанной и реальной цены
                self.mistakePredictionPercent = float("{:.2f}".format(
                    float((abs(self.devastorPredictFutureArray[len(self.devastorPredictFutureArray) - 1 - self.PREDICT_LIMIT] -
                           self.devastorLastTrade)
                           / self.devastorLastTrade)
                           * 100)
                                            ))
            except Exception as e:
                print("ERROR:", e)

            try:
                # считаем процент расхождения реальной цены и цены self.PREDICT_LIMIT минут назад
                self.mistakePricePercent = float("{:.2f}".format(
                    float((abs(
                        self.devastorPricePredictArr[len(self.devastorPricePredictArr) - 1 - self.PREDICT_LIMIT] -
                        self.devastorLastTrade)
                           / self.devastorLastTrade)
                          * 100)
                ))
            except Exception as e:
                print("ERROR:", e)

            try:
                # считаем качество оценки
                self.predictQuality = float("{:.2f}".format(
                    float((abs(
                        self.mistakePricePercent -
                        self.mistakePredictionPercent)
                           / self.mistakePricePercent)
                          * 100)
                ))

            except Exception as e:
                print("ERROR:", e)

            try:
                print("Текущая цена:",
                      int(self.devastorLastTrade),
                      '[изменение:',
                      str(self.mistakePricePercent) + '%]',
                      '<' + self.devastorRealTrend + '>',
                      '(прогноз был:',
                      str(int(self.devastorPredictFutureArray[
                              len(self.devastorPredictFutureArray) - 1 - self.PREDICT_LIMIT])) +
                      ')',
                      '<' + self.devastorPredictedTrend + '>',
                      '(погрешность:',
                      str(self.mistakePredictionPercent) + '%)',
                      'тренд предсказан:', self.trendPredicted,
                      'уровень предсказания:', str(int(self.trendPredictionPercent)) + '%)')
            except Exception as e:
                print("ERROR:", e)
                print("Текущая цена:",
                      int(self.devastorLastTrade),
                      '(прогноз: невозможен)')

            (self.devastorSellValue, self.devastorBuyValue) = self.devastorCalculateIdea(self.devastorTradeArray)

            # проверяем, если пришли нули, то апроксимиурем их до маленького значения
            #if self.devastorBuyValue == 0:
            #    self.devastorBuyValue = 0.001
            #if self.devastorSellValue == 0:
            #    self.devastorSellValue = 0.001

            # кладем пришедшие сигналы в соответствуюшие массивы
            self.devastorPricePredictArr.append(self.devastorLastTrade)
            self.devastorSellPredictArr.append(self.devastorSellValue)
            self.devastorBuyPredictArr.append(self.devastorBuyValue)

            if len(self.devastorPricePredictArr) > self.PREDICT_LIMIT:
                # кладем "подборку" за self.PREDICT_LIMIT минут ранее (если есть) в вектор вопросов нейросети
                self.X.append([self.devastorBuyPredictArr[0],
                               self.devastorSellPredictArr[0],
                               self.devastorPricePredictArr[0],
                               self.devastorLastBuyArr[0],
                               self.devastorLastSellArr[0],
                               self.devastorLastHighArr[0],
                               self.devastorLastLowArr[0],
                               self.devastorLastAvgArr[0],
                               self.devastorLastVolArr[0],
                               self.devastorLastVolCurrArr[0]
                               ])

                # кладем текущую цену в вектор ответов нейросети
                self.Y.append(self.devastorLastTrade)

                try:
                    with open('devastorDataset'+ self.devastorRandomSeed +'.csv', "a", newline="") as file:
                        writer = csv.writer(file)
                        # пишем в csv данные:
                        # абсцисса|покупки|продажи|цена|последняя покупка|последняя продажа|верхняя|нижняя|средняя|количество|общак|текущая|прогноз|тренд
                        writer.writerow([
                          self.abscissStr,
                          self.devastorBuyPredictArr[0],
                          self.devastorSellPredictArr[0],
                          self.devastorPricePredictArr[0],
                          self.devastorLastBuyArr[0],
                          self.devastorLastSellArr[0],
                          self.devastorLastHighArr[0],
                          self.devastorLastLowArr[0],
                          self.devastorLastAvgArr[0],
                          self.devastorLastVolArr[0],
                          self.devastorLastVolCurrArr[0],
                          self.devastorLastTrade,
                          str(int(self.devastorPredictFutureArray[
                              len(self.devastorPredictFutureArray) - 1 - self.PREDICT_LIMIT])),
                          self.trendPredicted
                        ])

                except Exception as e:
                    print("ERROR:", e)

                self.devastorNormedTrainX = self.devastorScaler.fit_transform(self.X)

                trainedAndPredict = 0
                # учим модель на этих данных (они дополняются каждую минуту)
                try:
                    self.devastorModel.fit(np.array(self.devastorNormedTrainX), np.array(self.Y))
                    print('Модель успешно обучена')
                    print('Точность обучения:', self.devastorModel.score(np.array(self.devastorNormedTrainX), np.array(self.Y)))
                    trainedAndPredict += 1
                except Exception as e:
                    print("ERROR:", e)
                    print('Для обучения недостаточно данных... ')
                # делаем прогноз на self.PREDICT_LIMIT минут вперед
                try:
                    # получаем и нормируем текущие данные для предсказания
                    self.devastorNormedTestX = self.devastorScaler.transform([[self.devastorBuyPredictArr[-1],
                                 self.devastorSellPredictArr[-1],
                                 self.devastorPricePredictArr[-1],
                                 self.devastorLastBuyArr[-1],
                                 self.devastorLastSellArr[-1],
                                 self.devastorLastHighArr[-1],
                                 self.devastorLastLowArr[-1],
                                 self.devastorLastAvgArr[-1],
                                 self.devastorLastVolArr[-1],
                                 self.devastorLastVolCurrArr[-1]
                               ]]
                    )
                    # кладем в основной массив предсказаний цен текущее предсказание по нормированным данным
                    self.devastorPredictFutureArray.append(self.devastorModel.predict(np.array(self.devastorNormedTestX)))

                    # кладем в переменную текущего предсказания последний элемент основного массива предсказания цен
                    self.devastorLastPredictedTrade = int(self.devastorPredictFutureArray[len(self.devastorPredictFutureArray) - 1])

                    print('Прогноз цены на', self.PREDICT_LIMIT + 1, 'минут:', self.devastorLastPredictedTrade)

                    if self.trendPredictionPercent > 60.0:
                        self.devastorTrendPredictionCounter += 1

                    # =====================================
                    # ======DDDDDD====DDDDD==DDDDDDD=======
                    # ======D========D=====D====D==========
                    # ======D========D=====D====D==========
                    # ======DDDDDD===D=====D====D==========
                    # ======D=====D==D=====D====D==========
                    # ======D=====D==D=====D====D==========
                    # ======DDDDDD====DDDDD=====D==========
                    # =====================================
                    # Тут будет торговать виртуальный бот
                    if len(self.devastorPredictFutureArray) > 0 and\
                            self.devastorTrendPredictionCounter > 10 and\
                            len(self.devastorPricePredictArr) > 2 * self.PREDICT_LIMIT:
                        self.devastorTradingStarted = True

                    if self.devastorTradingStarted:
                        # тренд восходящий
                        if self.devastorLastPredictedTrade > self.devastorLastTrade:
                            if self.devastorVirtualBalance['USDT'] > 0:
                                self.devastorAmount = self.devastorVirtualBalance['USDT'] / self.devastorLastTrade

                                print("Покупка:", self.devastorAmount, 'BTC',
                                      'по цене', self.devastorLastTrade, 'USDT',
                                      'за', self.devastorVirtualBalance['USDT'], 'USDT')

                                self.devastorVirtualBalance['USDT'] = 0
                                self.devastorVirtualBalance['BTC'] = self.devastorAmount
                            else:
                                print('Не хватает USDT для покупки BTC...')

                        # тренд нисходящий
                        if self.devastorLastPredictedTrade < self.devastorLastTrade:
                            if self.devastorVirtualBalance['BTC'] > 0:
                                self.devastorAmount = self.devastorVirtualBalance[
                                                          'BTC'] * self.devastorLastTrade

                                print("Продажа:", self.devastorVirtualBalance[
                                                          'BTC'], 'BTC',
                                      'по цене', self.devastorLastTrade, 'USDT',
                                      'за', self.devastorVirtualBalance['USDT'], 'USDT')

                                self.devastorVirtualBalance['BTC'] = 0
                                self.devastorVirtualBalance['USDT'] = self.devastorAmount
                            else:
                                print('Нет доступных BTC для продажи...')

                        # вывод текущей статистики
                        self.devastorProfit = self.devastorVirtualBalance['BTC'] * self.devastorLastTrade +\
                                              self.devastorVirtualBalance['USDT'] - self.devastorStartBalance
                        print('Текущий баланс - USDT:', self.devastorVirtualBalance['USDT'],
                              'BTC:', self.devastorVirtualBalance['BTC'],
                              '| профит:', self.devastorProfit, 'USDT')

                        l = open('devastorBalanceLog'+ self.devastorRandomSeed +'.txt', 'a', encoding='utf-8')
                        print(datetime.now().strftime('%H:%M:%S'), self.devastorVirtualBalance['USDT'], file=l)
                        l.close()


                    trainedAndPredict += 1


                except Exception as e:
                    print("ERROR:", e)
                    self.devastorLastPredictedTrade = 0
                    print('Для прогноза недостаточно данных... ')

                # сдвигаем влево массивы покупки продажи и цен
                self.devastorPricePredictArr.pop(0)
                self.devastorSellPredictArr.pop(0)
                self.devastorBuyPredictArr.pop(0)
                # сдвигаем влево массивы всех сигналов
                self.devastorLastBuyArr.pop(0)
                self.devastorLastSellArr.pop(0)
                self.devastorLastHighArr.pop(0)
                self.devastorLastLowArr.pop(0)
                self.devastorLastAvgArr.pop(0)
                self.devastorLastVolArr.pop(0)
                self.devastorLastVolCurrArr.pop(0)
