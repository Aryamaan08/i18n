# اكتب String (اختياري) - أحد ما يلي: المهام - سيتم وضع العناصر في هذه الفئة في فئة المهام القياسية. يمكن أن يكون هناك فئة واحدة فقط ، وسيتم عرضها دائمًا في أسفل قائمة الانتقال السريع. متكرر - يعرض قائمة من الملفات التي يفتحها التطبيق بشكل متكرر ، يتم تعيين اسم الفئة وعناصرها بواسطة Windows. حديث - يعرض قائمة بالملفات التي تم فتحها مؤخرًا بواسطة التطبيق ، يتم تعيين اسم الفئة وعناصرها بواسطة Windows. يمكن إضافة العناصر إلى هذه الفئة بشكل غير مباشر باستخدام app.addRecentDocument (مسار). custom - لعرض المهام أو روابط الملفات ، يجب أن يتم تعيين الاسم بواسطة التطبيق. اسم السلسلة (اختياري) - يجب تحديده إذا كان النوع مخصصًا ، وإلا يجب حذفه. عناصر JumpListItem [] (اختياري) - صفيف كائنات JumpListItem إذا كان النوع هو مهام أو مخصص ، وإلا يجب حذفه. ملاحظة: إذا كان كائن JumpListCategory لا يحتوي على النوع ولا خاصية الاسم ، فسيتم افتراض أن نوعه هو مهام. إذا تم تعيين خاصية الاسم ولكن تم حذف خاصية الكتابة ، فيتم افتراض أن النوع مخصص.

* `type` String (optional) - One of the following:
  * `tasks` - سيتم وضع عناصر هذه الفئة في الفئة`Tasks`المعيارية. يمكن أن يكون هناك فئة واحدة فقط ، وسيتم عرضها دائمًا في أسفل قائمة الانتقال السريع.
  * `frequent`- يعرض قائمة من الملفات التي يتم فتحها بشكل متكرر من قبل التطبيق ، يتم تعيين اسم الفئة وعناصرها بواسطة Windows.
  * `recent` - يعرض قائمة بالملفات التي تم فتحها مؤخرًا بواسطة التطبيق ، ويتم تعيين اسم الفئة وعناصرها بواسطة Windows. يمكن إضافة عناصر لهذه الفئة بطريقة غير مباشرة باستخدام `app.addRecentDocument(path)`.
  * `custom` - يعرض المهام أو روابط الملفات, `name` يجب أن يحددها التطبيق.
* `name` السلسلة (اختياري) - يجب تحديدها إذا`type` هو `custom`, وإلا يجب أن يكون.
* ` items </ 0> JumpListItem [] (اختياري) - صفيف الكائنات <a href="jump-list-item.md"><code> JumpListItem </ 1> إذا كانت <code> type </ 0> هي <code> مهمات </ 0> أو <code> custom < / 0> ، وإلا يجب حذفها.</p></li>
</ul>

<p spaces-before="0"><strong x-id="1">Note:</strong> إذا كان الكائن <code>JumpListCategory` ليس لديه `type` ولا `name` تعيين الخواص `type` يفترض أن يكون `tasks`. إذا كانت خاصية `name` معينة لكن خاصية `type` يتم حذفها ويفترض أن يكون `type` `custom`.
