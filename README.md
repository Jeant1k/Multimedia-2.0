# Лабораторные работы по Машинному Обучению и Компьютерному Зрению

Этот репозиторий содержит результаты выполнения трех лабораторных работ, посвященных исследованию и применению различных моделей машинного обучения в области компьютерного зрения. Каждая лабораторная работа представлена в отдельном модуле (директории) и включает Jupyter Notebook с кодом, подробными комментариями к каждой ячейке и выводами.

## Лабораторная работа №7: Проведение исследований с моделями семантической сегментации

**Цель:** Изучить и применить на практике модели семантической сегментации из библиотеки `segmentation_models.pytorch` (SMP), провести их сравнение, попытаться улучшить их качество, а также реализовать собственную модель сегментации.

**Набор данных:** Oxford-IIIT Pet Dataset (сегментация основного объекта, фона и границ – 3 класса).

**Метрики качества:** Pixel Accuracy, Mean Intersection over Union (mIoU), Dice Coefficient.

**Основные этапы и модели:**
1.  **Baseline (SMP):**
    *   Модель: `U-Net` с энкодером `ResNet34`.
    *   Результаты (Test): mIoU: `0`, Pixel Accuracy: `0.8528`.
2.  **Улучшение Baseline (SMP):**
    *   Модель: `FPN` с энкодером `EfficientNet-B0`.
    *   Техники: Продвинутые аугментации (`albumentations`), оптимизатор `AdamW`, планировщик `CosineAnnealingLR`.
    *   Результаты (Test): mIoU: `0`, Pixel Accuracy: `0.8497`. *Выбранные улучшения не привели к положительному эффекту, а даже ухудшили некоторые показатели.*
3.  **Custom U-Net (собственная реализация):**
    *   **С базовыми настройками:**
        *   Результаты (Test): mIoU: `0`, Pixel Accuracy: `0.7920`. *Значительно уступает SMP U-Net, что ожидаемо из-за отсутствия предобученного энкодера.*
    *   **С улучшенными настройками (аналогично п.2):**
        *   Результаты (Test): mIoU: `0`, Pixel Accuracy: `0.8122`. *Незначительное улучшение кастомной модели, но все еще сильно уступает SMP моделям.*

**Ключевые выводы из Лабораторной работы №7:**
*   **Проблема нулевого mIoU:** Во всех экспериментах метрика mIoU оставалась на нуле, что указывает на неспособность моделей корректно сегментировать все три класса одновременно. Вероятно, модели предсказывают преимущественно доминирующий класс, что дает высокую Pixel Accuracy, но низкий mIoU.
*   Значение предобученных энкодеров для SMP моделей.
*   Применение стандартных техник "улучшения" не гарантирует положительный результат без тщательной настройки и достаточного времени обучения, особенно при наличии системных проблем, как нулевой mIoU.
*   Требуется дальнейшее исследование причин нулевого mIoU: проверка реализации метрики, увеличение эпох, эксперименты с функциями потерь, анализ предсказаний по классам.

**Технологии:** PyTorch, `segmentation_models.pytorch`, `albumentations`, `torchvision`.

## Лабораторная работа №8: Проведение исследований с моделями обнаружения и распознавания объектов (Ultralytics YOLO)

**Цель:** Изучить и применить на практике модели обнаружения объектов семейства YOLO (Ultralytics), провести их сравнение, попытаться улучшить их качество и сравнить подходы fine-tuning и обучения "с нуля".

**Набор данных:** COCO128 (поднабор COCO с 80 классами).

**Метрики качества:** Precision, Recall, mAP@0.5, mAP@0.5:0.95.

**Основные этапы и модели (YOLOv8):**
1.  **Baseline (Fine-tuning):**
    *   Модель: `yolov8n.pt` (предобученная).
    *   Эпохи: ~25 (N).
    *   Результаты (Val): mAP@0.5:0.95: `0.6047`.
2.  **Улучшение Baseline (Fine-tuning):**
    *   Модель: `yolov8s.pt` (более крупная, предобученная).
    *   Эпохи: ~40 (M, M > N).
    *   Результаты (Val): mAP@0.5:0.95: `0.7671`. *Значительное улучшение (+26.86%) благодаря более мощной архитектуре и большему числу эпох.*
3.  **"Кастомная" реализация (Обучение "с нуля"):**
    *   **Модель `yolov8n.yaml` (с нуля):**
        *   Эпохи: ~40 (M).
        *   Результаты (Val): mAP@0.5:0.95: `0.0010`. *Катастрофическое падение качества по сравнению с fine-tuning (-99.83%).*
    *   **Модель `yolov8s.yaml` (более крупная, с нуля):**
        *   Эпохи: ~40 (M).
        *   Результаты (Val): mAP@0.5:0.95: `0.0018`. *Также крайне низкое качество (-99.77% по сравнению с `yolov8s.pt` fine-tune), подтверждая неэффективность обучения с нуля на малых данных.*

**Ключевые выводы из Лабораторной работы №8:**
*   **Критическая важность Transfer Learning (Fine-tuning):** Дообучение предобученных моделей (`.pt`) на малых датасетах (COCO128) дает на порядки лучшие результаты, чем обучение "с нуля" (`.yaml`).
*   **Влияние размера модели и эпох при fine-tuning:** Использование более крупных архитектур (YOLOv8s vs YOLOv8n) и увеличение числа эпох обучения при дообучении приводит к существенному повышению качества.
*   **Неэффективность обучения "с нуля" на малых датасетах:** Модели, обученные с нуля даже с увеличенным числом эпох и более крупной архитектурой, не способны показать приемлемое качество на датасете COCO128.
*   COCO128 удобен для быстрых экспериментов, но его размер делает обучение "с нуля" нерепрезентативным для реальных задач.

**Технологии:** PyTorch, `ultralytics` (YOLOv8).

## Общие выводы по всем работам
*   **Важность предобучения (Transfer Learning):** Эксперименты последовательно демонстрируют, что использование весов моделей, предобученных на больших датасетах (например, ImageNet для энкодеров сегментационных моделей или COCO для моделей детекции), является ключевым фактором успеха, особенно при работе с ограниченными целевыми наборами данных.
*   **Подбор архитектуры и гиперпараметров:** Выбор подходящей архитектуры модели и тщательная настройка гиперпараметров (скорость обучения, оптимизатор, аугментации, количество эпох) играют значительную роль в достижении оптимальных результатов. Однако, как показала ЛР№7, не всегда более сложные подходы приводят к немедленному улучшению, особенно если есть фундаментальные проблемы с обучением или метриками.
*   **Анализ результатов и метрик:** Критически важно не только получать численные значения метрик, но и понимать их значение, а также анализировать предсказания моделей визуально для выявления проблем и потенциальных путей улучшения. Нулевое значение mIoU в ЛР№7 является ярким примером необходимости глубокого анализа.
*   **Практический опыт:** Выполнение данных лабораторных работ позволило получить практический опыт в полном цикле разработки и исследования моделей компьютерного зрения для задач семантической сегментации и детекции объектов, включая подготовку данных, выбор и обучение моделей, оценку их качества и интерпретацию результатов.
