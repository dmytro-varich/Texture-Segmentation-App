# Texture-Segmentation-App


![image](https://github.com/user-attachments/assets/bb125021-b80e-4e4c-aaf8-66801e9b439e)

> This task involves texture segmentation of grayscale images from the Prague dataset. It is part of a Computer Vision course at my university. As a student, I find it quite challenging to fully understand this area, so I decided to replicate an algorithm published in this article - [A DATA-CENTRIC APPROACH TO UNSUPERVISED TEXTURE SEGMENTATION USING PRINCIPLE REPRESENTATIVE PATTERNS](https://www.researchgate.net/publication/331731944_A_DATA-CENTRIC_APPROACH_TO_UNSUPERVISED_TEXTURE_SEGMENTATION_USING_PRINCIPLE_REPRESENTATIVE_PATTERNS?enrichId=rgreq-7c9d63d6e208de4d9a21b1faf1fb37b1-XXX&enrichSource=Y292ZXJQYWdlOzMzMTczMTk0NDtBUzo3NDk0NTY4ODI0MjE3NjBAMTU1NTY5NTg1MzQzNg%3D%3D&el=1_x_3&_esc=publicationCoverPdf).
I would be incredibly grateful if a professional in this field could help identify and correct my mistakes, as well as provide recommendations for improvement.
My code is available [here on GitHub](https://github.com/dmytro-varich/Texture-Segmentation-App/blob/main/segmentation_main.ipynb).


1. **Pokusy o zlepšenie algoritmu**  
   Strávil som celé prázdniny snahou vylepšiť algoritmus. Odstránil som veľa zbytočných častí a urobil niekoľko vylepšení, no, žiaľ, presnosť segmentácie sa nezvýšila. Napriek tomu je navrhnutý prístup založený na **adaptabilite**. To znamená, že algoritmus sa automaticky prispôsobuje rôznym textúram a efektívne ich spracováva bez potreby manuálneho nastavovania parametrov pre každé konkrétne obrázok. Tento prístup umožňuje vyhnúť sa závislosti od pevne nastavených filtrov a robí algoritmus univerzálnym.  

2. **Hlavná myšlienka algoritmu**  
   Hlavným cieľom algoritmu je extrahovať textúrne črty zo všetkých obrázkov v datasete. Tieto črty zahŕňajú:  
   - **Entropiu** (na vyhodnotenie nehomogenity a náhodnosti textúry),  
   - **Medián** (na určenie celkovej úrovne intenzity),  
   - **Laplacián** (na zvýraznenie okrajov a drobných detailov),  
   - **Wavelety** (na analýzu textúr v rôznych mierkach),  
   - **Fourierovu transformáciu** (na analýzu frekvenčných charakteristík textúr).  

3. **Rozdelenie obrázkov na patche**  
   Po extrahovaní textúrnych čŕt sa každý obrázok rozdelí na malé časti (patche), aby sa získali lokálne charakteristiky textúry. Patche umožňujú zohľadniť lokálne aj globálne vlastnosti textúr.  

4. **Klasifikácia a reprezentatívne vzory**  
   Zo všetkých textúrnych patchov sa pomocou metódy \( K \)-means klasifikácie identifikujú kľúčové **reprezentatívne vzory** (PRP). Tieto vzory reprezentujú hlavné textúrne charakteristiky obrázkov a slúžia ako šablóny na porovnávanie. Prostredníctvom výpočtu podobnosti každého patchu s PRP sa určujú textúry, ku ktorým patria.  

5. **Metóda `compute_texture_features`**  
   Táto metóda vykonáva niekoľko hlavných krokov:  
   - **Extrahovanie čŕt**: Najprv sa opakuje proces extrahovania textúrnych čŕt, rozdelenia na patche a ich transformácie na vektory, ktoré opisujú lokálne charakteristiky textúry.  
   - **Porovnávanie patchov s centroidmi klasifikácie (PRP)**: Patche sa porovnávajú s PRP pomocou vzorca podobnosti založeného na Gaussovom jadre:  
     \[
     w(P_i, P_j) = \frac{1}{Z(i)} \exp\left(-\frac{\|P_i - P_j\|_2^2}{h^2 \cdot \sigma^2}\right)
     \]  
     **Vysvetlenie parametrov**:  
     - \( \|P_i - P_j\|_2^2 \): Euklidovská vzdialenosť medzi patchom a vzorom, ktorá meria ich podobnosť.  
     - \( h \): Parameter vyhladzovania, ktorý reguluje, ako rýchlo klesá váha so zvyšujúcou sa vzdialenosťou.  
     - \( \sigma \): Mierka, ovplyvňujúca citlivosť na vzdialenosť.  
     - \( Z(i) \): Normalizačná konštanta:  
       \[
       Z(i) = \sum_j \exp\left(-\frac{\|P_i - P_j\|_2^2}{h^2 \cdot \sigma^2}\right)
       \]  
       Táto konštanta zabezpečuje, že všetky váhy \( w(P_i, P_j) \) sa sčítajú na 1.  

   **Význam vzorca**:  
   - **Objektívnosť**: Gaussovo jadro vyhladzuje šum a zdôrazňuje najbližšie vzory.  
   - **Adaptabilita**: Vzorec sa automaticky prispôsobuje rôznym textúram vďaka normalizácii.  
   - **Kompaktnosť**: Namiesto uchovávania všetkých čŕt sa ukladá iba informácia o podobnosti s PRP, čo zjednodušuje ďalšie spracovanie.  

6. **Zosilnenie čŕt a filtrovanie**  
   Po výpočte podobnosti sa textúrne črty zosilnia pomocou kontrastných váh, aby sa zvýraznili najvýraznejšie vzory. Slabé vzory sa následne filtrujú pomocou priemernej pravdepodobnosti (\( \text{mean\_probabilities} \)), čo umožňuje sústrediť sa na najvýznamnejšie vzory.  

7. **Priradenie klastrov**  
   Každý patch sa priradí klastru, s ktorým má najvyššiu podobnosť:  
   \[
   \text{cluster\_id} = \arg\max_j w(P_i, P_j)
   \]  
   Výsledky segmentácie sa prevedú na textúrnu mapu, kde každý klaster má jedinečný identifikátor.  

8. **Hlavné problémy**  
   Algoritmus niekedy vytvára viac textúrnych segmentov, než je potrebné, kvôli nedostatočne prísnemu prahu. Tento problém riešime dodatočným filtrovaním slabých vzorov.  

9. **Postprocesing**  
   Po klasifikácii sa aplikuje postprocesing:  
   - **Mediánový filter**: Vyhladzuje šum.  
   - **Morfologické operácie**: Odstraňujú medzery medzi segmentmi a zlepšujú celkovú kvalitu.  

---

1. **Попытки улучшения алгоритма**  
   Я потратил целые каникулы, чтобы попытаться улучшить алгоритм. Убрал много лишнего и внес небольшие изменения, но, к сожалению, точность сегментации не выросла. Тем не менее, предложенный подход рассчитан на **адаптивность**. Это означает, что алгоритм автоматически подстраивается под различные текстуры, эффективно обрабатывая их без необходимости ручного выбора параметров для каждого конкретного изображения. Такой подход позволяет избежать зависимости от жестко заданных фильтров и делает алгоритм универсальным.

2. **Основная идея алгоритма**  
   Основная идея заключается в извлечении текстурных признаков из всех изображений в датасете. Эти признаки включают:  
   - **Энтропию** (для оценки неоднородности и случайности текстуры);  
   - **Медиану** (для определения общего уровня интенсивности);  
   - **Лапласиан** (для выделения границ и мелких деталей);  
   - **Вейвлеты** (для анализа текстур на различных масштабах);  
   - **Фурье-преобразование** (для анализа частотных характеристик текстур).  

3. **Разделение изображения на патчи**  
   После извлечения текстурных признаков каждое изображение разбивается на небольшие патчи (локальные области), чтобы получить локальные особенности текстуры. Патчи позволяют учитывать как локальные, так и глобальные характеристики текстур.

4. **Кластеризация и репрезентативные паттерны**  
   Из набора всех текстурных патчей методом кластеризации \( K \)-средних выделяются ключевые **репрезентативные паттерны** (PRP). Эти паттерны представляют основные текстурные особенности изображения и используются как эталонные шаблоны для сравнения. Путем вычисления сходства каждого патча с PRP определяются текстуры, к которым он относится.  

5. **Метод `compute_texture_features`**  
   Этот метод выполняет несколько ключевых шагов:  
   - **Получение признаков**: Сначала мы повторяем процесс извлечения текстурных признаков, разделения их на патчи и преобразования патчей в векторы, которые описывают локальные текстурные характеристики.  
   - **Сравнение патчей с центроидами кластеров (PRP)**: Патчи сравниваются с PRP с помощью формулы сходства, основанной на Гауссовом ядре:  
     \[
     w(P_i, P_j) = \frac{1}{Z(i)} \exp\left(-\frac{\|P_i - P_j\|_2^2}{h^2 \cdot \sigma^2}\right)
     \]  
     **Расшифровка параметров**:  
     - \( \|P_i - P_j\|_2^2 \): Евклидово расстояние между патчем и паттерном, определяющее степень их сходства.  
     - \( h \): Параметр сглаживания, регулирующий, насколько быстро вес убывает при увеличении расстояния.  
     - \( \sigma \): Масштаб, влияющий на чувствительность к расстоянию.  
     - \( Z(i) \): Нормировочная константа:  
       \[
       Z(i) = \sum_j \exp\left(-\frac{\|P_i - P_j\|_2^2}{h^2 \cdot \sigma^2}\right)
       \]  
       Эта константа обеспечивает, что все веса \( w(P_i, P_j) \) суммируются до 1.  

   **Значимость формулы**:  
   - **Объективность**: Гауссово ядро сглаживает шум и выделяет наиболее близкие паттерны.  
   - **Адаптивность**: Формула автоматически подстраивается под различные текстуры благодаря нормировке.  
   - **Компактность**: Вместо хранения всех признаков используется только информация о сходстве с PRP.  

6. **Усиление признаков и фильтрация**  
   После вычисления сходства текстурные признаки усиливаются с использованием контрастных весов, чтобы выделить наиболее выраженные паттерны. Затем слабые паттерны отфильтровываются с использованием среднего значения вероятностей (\( \text{mean\_probabilities} \)). Это позволяет сосредоточиться на наиболее значимых паттернах.  

7. **Назначение кластеров**  
   Каждый патч присваивается кластеру, с которым у него наибольшее сходство:  
   \[
   \text{cluster\_id} = \arg\max_j w(P_i, P_j)
   \]  
   Результаты сегментации преобразуются в текстурную карту, где каждый кластер отображается уникальным идентификатором.  

8. **Основные проблемы**  
   Иногда алгоритм создает больше текстурных сегментов, чем нужно, из-за недостаточно строгого порога. Для борьбы с этим используется дополнительная фильтрация слабых паттернов.  

9. **Постобработка**  
   После классификации применяется постобработка:  
   - **Медианный фильтр**: Сглаживает шумы.  
   - **Морфологические операции**: Устраняют разрывы между сегментами.  

---

