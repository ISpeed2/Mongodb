# Занятие 4 План запроса и индексы

Практикум выполнен на локальной MongoDB 8.3.11 в базе <code>logs</code>, коллекции <code>events</code>.

## Среда

- MongoDB: <code>mongodb://127.0.0.1:27017</code>
- База: <code>logs</code>
- Коллекция: <code>events</code>
- Документов в коллекции: 1200
- Документов с <code>level: "error"</code>: 180
- Инструменты: <code>mongosh</code> и Paste the Plan
- Планы получены командой <code>explain("executionStats")</code>

## Сводная таблица планов

Шаги указаны снизу вверх: от чтения данных к итоговой операции.

| Файл | Индекс | nReturned | totalDocsExamined | totalKeysExamined | Шаги плана |
|---|---|---:|---:|---:|---|
| [plan-1.json](plan-1.json) | нет | 33 | 1200 | 0 | <code>COLLSCAN → SORT</code> |
| [plan-2.json](plan-2.json) | <code>service, level, ts, duration_ms</code> | 33 | 33 | 43 | <code>IXSCAN → FETCH</code> |
| [plan-3a.json](plan-3a.json) | <code>duration_ms, service, level, ts</code> | 33 | 33 | 378 | <code>IXSCAN → SORT → FETCH</code> |
| [plan-3b.json](plan-3b.json) | <code>service, level, duration_ms, ts</code> | 33 | 33 | 33 | <code>IXSCAN → SORT → FETCH</code> |
| [plan-4-bez.json](plan-4-bez.json) | нет | 10 | 1200 | 0 | <code>COLLSCAN → SORT</code> |
| [plan-4.json](plan-4.json) | <code>level, duration_ms</code> | 10 | 10 | 10 | <code>IXSCAN → FETCH → LIMIT</code> |

## Задача 1 Запрос без индекса

Запрос:

~~~~javascript
db.events.find({
  service: "payments",
  level: "error",
  duration_ms: { $gt: 500 }
}).sort({ ts: -1 })
~~~~

Запрос вернул 33 документа, но сервер просмотрел все 1200 документов и не просмотрел ни одного ключа индекса. План состоит из <code>COLLSCAN</code> и <code>SORT</code>: подходящего индекса нет, поэтому MongoDB перебирает коллекцию целиком, а затем сортирует найденные документы в памяти.

[Ссылка на разбор plan-1](https://dfrancour.dev/tools/mongodb-paste-the-plan#v1:zVZtb-JGEP4vq_tIIhvbBPPNGGijhiPFtKfqVFWLd4At9q5vd03gEP-9s2veklNy0VWq-olk3ud5Zna8J7CtCsrF76A0l4L0iE9a5EsNavdYUCFAkd6eCFqCrmgOqC_kUt_CBoTRaFlRpYH9au2t4QcqGOl93pMCLQongS_oBEpJRQ6H1p5oUBtuI510Fd2VLppTs1pRg5X8VWpnsjSkF3ne4fDnoUW4YLAd8cKAygAVC1poOJb7M9UrjDb0krtO3I1tbdhASvMVZCtawRv6XwCLJ2k_SYZh2EaNrAwv-VdXyIyXMOZFwbEer0VKur23VQCbqEwWtTXRU7Bh2Lmgi1Ei2FtWWU6FnskhkiAZvDSoVC2AZVhKQVUTUJ-VT1wILpaWJQsU1-lzZ23o0vKVTaYzbElLZR6pQeScOcLdu_ER0hLKB0yAYPpe2I3uOh42aXaVddW8rAogFveqNlkTcH-OnE4eHrI0-YgGC8fJfzABQutvZpBxBblppnch1RNVzEYiCv5GOTALEXp9tv4IYe7YwG6Mi3-R1HkONrxRNeInpmBqJSyiQXDleD0PbYRKGlrgAOnhlpbcmXtH6UDmV1K_bZG9zr8E_R7mvl_IUOO4UgMu9ZNUa23TBVGLULahIj-5CgBmvVwxfvP_HxyKpmZNN2BRacIo0EaqKwGmmYzQ81-OUoMMNTTjXyHDSLa42O9i3BpfkgHX6wsQFbbXrJ37E9gUcqnYM1F_Z-CZwAXH2hFEm8OpKqDrmaL5GtgYyqNLk_V_NNs_SLSl8lWi_c7dDxP93mVD6TejfrAb6B7m69c3SeLhKPL9yPe8NAgGcZDESRqF3XYUpnEn6EaRNwrTYTDoh_7AH3T8YdRBq1EYBOkoHGCqXJal42GPzNhfcq7twtSZhgv-rRNrR7par5PRjPiz2f7A5kcoiFVjeMAXeSGt0UpqNCZjmvelXN8kXN185Gtu6G0hc1rYS-PCte88H8nYnE9t9za49e29XXJzOcDz7rwbsA50wKfAwjiK7rqUdfPOIm4HIWVBO85jlF4KeaQKDzS23rwnwm4mLdxRHuHRNv16scCLictwnPyr_TxZ42NVW6AyWascflKyrsZ0i8si1e51L5cD7frY6RqvkV3oxuc3jYv03nR4zD8hl_JpZIdUvzexaw6NJ7XBHcaQb7f4gOzUlVv1eysqgXEc-lMpGOhtf5fzUckVn3Nz6ngMagkTMZZiKbNmf15gkzA2k9jh95qxHNqdTqUwStpBNbg8c_wq0EZxe8jIy1qaL7T7pcAFdt8Hn7hZpbIo3FyPpJrCEraYE0dFru3vPw)

## Задача 2 Индекс по правилу ESR

Разбор запроса:

- E — равенство: <code>service</code>, <code>level</code>;
- S — сортировка: <code>ts</code>;
- R — диапазон: <code>duration_ms</code>.

Создан индекс:

~~~~javascript
db.events.createIndex({
  service: 1,
  level: 1,
  ts: -1,
  duration_ms: 1
})
~~~~

После создания индекса сервер просмотрел 33 документа и 43 ключа индекса. Шаг <code>SORT</code> исчез, потому что порядок по <code>ts</code> обеспечивается индексом. Число просмотренных документов уменьшилось с 1200 до 33, то есть примерно в 36,4 раза.

[Ссылка на разбор plan-2](https://dfrancour.dev/tools/mongodb-paste-the-plan#v1:7VZLc-M2DP4rGc4eenAyUmTZlm_yq5tpHWcjb7edbMZDi7DNtUxqSSpx1uP_XpCSX9nmcWxnepIEAuAH4AOEDYF1nlEu_gCluRSkTXxSI98LUE83GRUCFGlviKAr0DlNAc8zOdcX8ADCaNTMqdLAPll9q_iBCkbadxuSoUbmJPAdjUApqch2W9sQDeqBW0-7s5w-rZw3d8wKRQ0imay0U5kb0g49b7u939YIFwzWA54ZUAngwYxmGiq4H6leoLe-FzcbUSuy2DCALk0XkCxoDq-c_wYInjRjr-714wBPZG74iv9wQMZ8BUOeZRzx-DWyousriwLYSCUyK6yKvgXrhu0BHZRiwV7TSlIq9Fj2sQiSwXOFXBUCWIJQMqpKh3p_-MiF4GJuq2QTxXX31FgbOrf1GvTH3Y8Yk9D6p-pxkRcmKRU3e4urP5NufP2CyRKebqjBArhb98XEzFQlxzdUbJ_j86SW_q5-10gmdFuZTvyJM8Sn0ZNzf3JkNLFc5HpYZIa7Gu0SVwkQyEKfwLi73-OwrxaIfZ4AuXNM0p8FR97sfXKdOC4fCW6oMpxmB4lFv2-US3TLFaSmbJuZVI9UMVKpdWQh2DNs5O7rnuxfSe3s-OueHCG3iq5jSq3q1am4iMjdkK4x_trZkAt8upPTGMkv2DS1My5meLjFziIKviFYYJYwuywgoVLHTeSAcWgPkiJNwdbfqAJjF7dgCiUsv4LgyPC4OzAjRhqaISLdX9MVd-r1oBL3ZHokPvHiKKjfR-M3gfQ1Ni81aORhl0i11CUIyh6oSHeGAoBZG9KOyo-_OGTM2Wj6ADYhpQcF2kh1JMAbRgPH83_qD_ZTlDRTQJmdT6Ppt9LDG233Lw_x_xHwHxwBrmzP2lIDWOb4njXN9RhpACU_7GdPyTyvvnOgy7Gi6RLYEFadJ2O71XNzxf18j_-wcRz1B6Hvh77ndYOgFwVxFHfDeusyrHejRtAKQ29Q7_aDXqfu9_xew--HDdQa1IOgO6j3MH-pXK3cJrEhM26fZE--mfv7n2T1sEHs81ctHLWX1wmMXipjZSVVUfCBTSuuE3uM7gH_ujNplRZSozIZ0rQj5fI85ur8mi-5oReZTJEgmCLn7rLp-c0aedivU62L4MK3JJ5zc1iypq1pK2ANaIBPgdWjMGy2KGuljVl0GdQpCy6jNELpAQgyEZsGQy_HpLDtRzO3eA1wMTOdYjbDrYj_gKo6vldvhc2G5-ZNqY0TuLCJSmShUvhVySJHFmFBpXp62crdgXodjHSJG0eCgZY2nzVOrvdehwvbF6ylfBzYKaTfe7ELDpVHhcGhiS5fD_F3rE6Ru9l6ZUUrYByn2g4KOnrd3t15o-SCT7nZRTwENYeRGEoxl0k1wU9zEzM2lhjhW8HYGtp53ZXCKGmJarB5prj5aaO4_T2T51jKLfxqLnBCux3wCzeLrswyx-uBVLcwh3U5XOXSPv8G)

## Задача 3 Порядок полей

### Вариант А Диапазон в начале

Индекс:

~~~~javascript
db.events.createIndex({
  duration_ms: 1,
  service: 1,
  level: 1,
  ts: -1
})
~~~~

Сервер просмотрел 378 ключей, чтобы вернуть 33 документа, и снова выполнил <code>SORT</code>. Диапазон по <code>duration_ms</code> находится первым, поэтому в индексный диапазон попадают события разных сервисов и уровней. Порядок по <code>ts</code> внутри такого диапазона не гарантирован.

[Ссылка на разбор plan-3a](https://dfrancour.dev/tools/mongodb-paste-the-plan#v1:7VdLc-M2DP4rGc4eenAyUiT5kZuf3UzXcRp5u-1kMxlGhG2uJVJLUom9Hv_3gpT8ynvaHtqZ-iKKBMAPwAcIXhFY5Cnl4jdQmktBzohPauR7AWp5mVIhQJGzFRE0A53TBPA8lVN9AvcgjEbJnCoN7FcrbwU_UMHI2fWKpCiRuh34jkqglFRkva6tiAZ1z62lzVlOl5mz5o5ZoahBJLeZdiJTQ84iz1uvb9Y1wgWDxYCnBlQMeDChqYYK7keqZ2it77Ub9VazZbGhA12azCCe0RxeOf8FEDxptL3Q67cDPJG54Rn_4YCMeQZDnqYc8Xg1ktHFuUUBbKRimRZWRF-BNcO2gHZCbcFek4oTKvRY9jEJksFjgVwVAliMUFKqSoN6e_jAheBiarNkA8V191BZGzq1-Rr0x92P6JPQ-kn2uMgLE5eCq61GPLoa46GWylxSg7F2F6DC2bGPScgg-4SQMPy-FzajRt3DsJhlblUZTGiRmhdNn_8ed9sXL6CZw3LvvgMi-LUdb3BdsQtXW1SOGhfIU4tip3rr31aKuHJq-DT69tjSnOshguUu_ZucVBsIZKafwLi-2cNhXyogdmncuUWiPwuOlNza5Dp2ZbK3cUmV4TTd7Vj02xo8rRHGFSSmrMiJVA9UMVKJdWQh2DPYyE9YJ7UjLiY35AAnuf66LbKvpHa0_-ZEN15YQVeppVS1dCLOO3I9pAsMTu1oyAU-8WSNvxpR8A3RArNk3IQByZo43iMJjIO72ymSBCwBjCrQeXEFplDCcjcI9hT3Kw9DYqShKd6q-wua8VK80az2ezLZ3w8O75-Cfl-NvImkr7EzUAOuFzxINUfDoY-ylN1TkWw0BQCzShZiq3z9g0PKnJqm92CDUhpRoI1Uext4yWjgyP1ckbAnjtJUAWW2_43uvpUWXivrf5mPf6_LlLmnhsb8B8RoyoHz60iLAr9LPa7nu2Tn6F3ZxN0S2BUkUrGDrc7SwMGGM47gMYb2DneUA52PFU3mwIaQVSrlrW-1vb8UfBffF4MfRv8owf7vwv_BLuzS9rgxagDHnrCOnhS5HiMRoGSIfe0pmefV-7OM9srm7qar_RGq3W71B5HvR77ndYOg1wrarXY3CpunUdht1YNmFHmDsNsPep3Q7_m9ut-P6ig1CIOgOwh7GMJEZpkbFVdkwu2TbPk3ceOdq55N7HYj4jZK1URZe3leLFvLQU_5wO4qupN1mRrAsWoirdBMahQmQ5p0pJwft7k6vuBzbuhJKhPkCMbImTtteH6jRu6383LzJDjxLY-n3Oym6LvmXTNgdaiDT4GFrShqNClrJvVJ6zQIKQtOW0kLd3dAkIxYN-h6-akStgJp6ibrAU7eplNMJjj2Yg-q0rPXFzfS-BUsbKBiWagEflayyJErmFGpli9ruTtQroOeznGktH201PmssX-99zqcyL9gLuXDwDYi_d6LnXMoPCoMtk40-bqLnzA7Re467LndyoBxbGwbKGjodX1356WSM37HzcbjIagpjMRQiqmMq0_oYWzajI0leviWMzaHtmt3pTBKWqIaLJ47HO21UdzOSOQxlvJv1vlUYJN2Q_4XbmZdmaaO1wOprmAKC7wTqSLn9vkn)

### Вариант Б Диапазон перед сортировкой

Индекс:

~~~~javascript
db.events.createIndex({
  service: 1,
  level: 1,
  duration_ms: 1,
  ts: -1
})
~~~~

Сервер просмотрел только 33 ключа, но <code>SORT</code> всё равно остался. Диапазон по <code>duration_ms</code> стоит перед полем сортировки <code>ts</code>, поэтому индекс не может полностью обеспечить требуемый порядок.

[Ссылка на разбор plan-3b](https://dfrancour.dev/tools/mongodb-paste-the-plan#v1:7VdLc9s2EP4rHkwOPdAe0tTTNz0bTyPLNZWmHcfjgYiVhIgEGAC0pWj037sAKUpy4se0PbQz1YUEsLv4dvfb5WpDYJUllIvfQGkuBbkgAfHI1xzU-jqhQoAiFxsiaAo6ozHgeSLn-gweQBiNkhlVGtivVt4KvqOCkYvbDUlQInE78BWVQCmpyHbrbYgG9cCtpd1ZRteps-aOWa6oQST3qXYic0Mu6r6_3d5tPcIFg9WQJwZUBHgwo4mGEu57qhdobeB3mo12q22xoQM9Gi8gWtAMXjj_BRA8aXb8mj_ohHgiM8NT_s0BmfAURjxJOOLxPZLS1aVFAWysIpnkVkTfgDXDKkB7oY5gL0lFMRV6IgeYBMngqUCmcgEsQigJVYVBXR0-ciG4mNss2UBx3TtW1obObb6Gg0nvPfoktP4ue1xkuYkKwU2lEY1vJniopTLX1GCs3QWocHEaYBJSSD8gJAx_4Nda9WbDx7CYdWZVGcxonphnTV_-HvU6V8-gWcL64L6KJ4G3YxO-HdED1xUqR40r5CmaLVXvg3uniM8DNVwZfX9qac71CMFyl_5dTsoNBLLQRzBu7yoc9vUIiN0w7mmR6I-CIyUrm1xHrkwONq6pMpwm-x2LvqrBczTPFcSmqMiZVI9UMVKKdWUu2BNs5PZzVUefiXdyuLojB8itoCvGQqp8dSLHHpGfsOq8Ey5m7tB5R25HdIXB8U5GXOATT7b484iCL4gWmCXjLgxI1tjxHklgHNz9Th7HYAlgVI7OixswuRKWu2F4oHhYeTbV0tAEb9WDFU15Je62-zJ-sn14_Rz020rkVSADjY2BGnCt4FGqJRputDxC2QMV8U5RADCrg6tasfqDQ8KckqYPYCNSmFCgjVQHG3jFeOjc_VGFsO_cpIkCymzzG0-_FBZequl_lYd_r8EUeaeGRvwbRGjKYQss1hw_SX2ul_tEZ-hb0b_dK7AbiKViR1vdtYGjDWccwWME7R3uKAO6nCgaL4GNIC1Viltf63h_KfQ2vM-F3v9HufV_9_0Pdl-XtictQQMsdZmuTE-QBVDQwy77SmZZuf4hl_2io7uR6nBu6nTag2E9COqB7_fCsN8OO-1Or15rnddrvXYjbNXr_rDWG4T9bi3oB_1GMKg3UGpYC8PesNbH-MUyTd18uCEzbp-kIt_MzXRHUd3PhVX8yjHSe35ILJrKUTd5x6Yl14k9RvOAs9RMWqGF1ChMRjTuSrk87XB1esWX3NCzRMZIEIyRM3fe9IOmRx6qIbl1Fp4FlsRzbvaj87Q1bYWsAQ0IKLBau15vtihrxY1Z-zysURaet-M27u6BIBOxaND14gMlbPnRxI3TQxy3TTefzXDWxe5TpuegI-6k8duX20BFMlcx_KxkniFRMKNSrZ_XcnegXBc9XeIcaTtoofNRY-d663U4hn_CXMrHoe1C-q0XO-dQeJwbbJpo8mUXP2B28sz11ku7lQLj2NV2UNDQy_ruzmslF3zKzc7jEag5jMVIirmMyk_ncWw6jE0keviaMzaHtl_3pDBKWqIaLJ4pzvPaKG4HI_IUS_Hf6nIusEO7yf4TN4ueTBLH66FUNzCHFd6JVJFL-_wT)

### Вывод

В рабочей базе следует оставить индекс по правилу ESR:

~~~~javascript
{ service: 1, level: 1, ts: -1, duration_ms: 1 }
~~~~

Вариант Б просматривает меньше ключей на учебном наборе, но требует сортировки в памяти. На большой коллекции это может привести к превышению лимита сортировки в 100 МБ.

## Задача 4 Самостоятельный подбор индекса

Запрос:

~~~~javascript
db.events.find({ level: "error" })
  .sort({ duration_ms: -1 })
  .limit(10)
~~~~

Без индекса MongoDB просмотрела все 1200 документов, хотя вернула только 10.

Подходящий индекс:

~~~~javascript
db.events.createIndex({
  level: 1,
  duration_ms: -1
})
~~~~

Здесь <code>level</code> — поле равенства, а <code>duration_ms</code> — поле сортировки. После создания индекса сервер просмотрел 10 ключей и 10 документов; план содержит <code>IXSCAN → FETCH → LIMIT</code>.

В коллекции всего 180 ошибок, но сервер прочитал только 10 документов, потому что индекс сразу располагает ошибки по <code>duration_ms</code> от большей длительности к меньшей, а <code>limit(10)</code> останавливает чтение после первых десяти подходящих документов.

[Ссылка на план без индекса](https://dfrancour.dev/tools/mongodb-paste-the-plan#v1:tVbbcuM2DP0Xzj46GUm-yXmTFanNNF6nkdudnZ1OhxZhmzVFakkqcTaTfy9I-Zq03rTTPtkGAR7g4IDwM4FNLSiXv4I2XElyRULSIV8b0E93gkoJmlw9E0krMDUtAc-FWppLeABpDXrWVBtgPzt_5yjwQLgvH-Ar-oLWSpOXlw7hksEm58KCLsCSqwUVBrZAP1KzQud-2M2S6-vU3YrQKS1XUKxoDWfOfwKEJUkvz0ZJnuGJqi2v-DdqsZgZr2DCheCGXAUdUtHNjcsC2FQXSjTOxdyDu4btEzo4JZKd8ypKKs1MZUifYvDaodaNBFZgKoLq9kKzP3zkUnK5dPw6qrhJT4ONpUvHdDG9n2FJRml7Ry0y591Zo311v1d440WI3FZQ3SISshoGvbg_HARYrXCWpFKN9PYOsU-1u9TwqhZAXEfqxhYt1PMeM53e3hZp8hEdFr5bZ7sqjXmjCMY1lLbV0kLpR6qZ99XwB9qBubIx6stvaENaSs8w5oHBiHCwNGUJ7nqrG-RE3oNttHQsuWL2bsc9DrFIZalAUZhsQyvu3YOt9VqVR9YwCoJT_CWY93Tj-4lkBiVILXjoR6XXLrUojDqEsgcqy12oBGAuyicTtr8_cxBtzoY-gGOlvUaDsUofGRBmmvuS_yd5tJxRSwv-DQrEcGlHcdTtkAYn_pqb9YGiGgtvh8x_BXYPpdLsxDR-snBi8JdjVUivw_BHNdD1TNNyDWwC1TakRf1P9HrUvPifdM_156h78Wn7gij41-177wih9Y2AX1xN_gk9fiejMAiyfpwEeZoHYZIkcZzm2SAdBFE0zNMkHaXdZJylcRr3gl4vi4dBPMyHvTgaBqMgGyJUqaqKSuYoXHD3Sfa5vWF5R2-rxL-WoNjpr0M-sPm2Xh8C-gHwgVwoF7hSBr3IhJZjpdYXCdcXH_maW3opVEmFe_g9BCYaDjvkYb-z4svuZegW15Lbwyabx_O4ywYwgJAC6436_WFMWVwOFqOo26OsG43KEVoPidxRjZsO62ufAumGigq_3XLcfnbcLBa4wFCtW2keTdTOG9-ZpkKyCtXoEn7QqqkndINqVvrp76M8BvqNsdI1Lgc3cW3MLwaV_l443K2fsGHqMXdKNO8F9sWh87SxOGR45fkSb7E7Te1n8caZKmAclb1LBS86H-8x77Ra8Tm3u4onoJcwlRMll6poh-QVNwljM4UVfq8Y10M3uamSVisnUYsTMsclbazmbgeR17m0f3VulhKn1K_rT9yuUiWEF3Ku9D0sYYOYKBW1dp9_Ag)

[Ссылка на разбор plan-4](https://dfrancour.dev/tools/mongodb-paste-the-plan#v1:7VZbc9o4FP4rHU0fScYGAiZvxsFbZkuSxnTbTprJCOsAKrLkSjIhzeS_75EMhJA2yV4e9mGfbEnfuX86OncEVqWgXP4B2nAlyTEJSYN8r0DfngsqJWhyfEckLcCUNAc8F2pmDmEJ0hpEllQbYB8c3gEFHgj38xa-Ixa0Vprc3zcIlwxWKRcWdAaWHE-pMLA29I6aOYKPwtYgPjlJnFY0ndB8DtmclvDM-e-AZknQ73TiTq-LJ6q0vOA_qMVgxryAEReCG3IcNEhBV0PnBbAznSlROYi5AKeGbR16AMWSPYfKcirNWA0wfYrBPqDUlQSWoSuC6lqh2R7ecCm5nLn8ulRxkzwWNpbOXKbfD0fDMcYkUI2NC1VJTFwYuGSWlc1q1N0Wng7GyTuES2OelOnnEsPPWRKf_kJkAbfn1GK95E5hwwZhlfbZvS5Q5iDc1PYUKeJ0ONx1eL2Duj5wlOJmVAnLfcE2WVxvoJm52TFyebVn5fLKWTEfJUe-bMW5yTz7djbOqbaciocd59mW2k1UyzXktib6VOkbqhlZw_qYX_bIDXL5tWbwV9J4s_29Ik_cI5cjusJAGm9GXOIXIff3jvUavqE5YK7UmziQCrlnFZbDensPO1WegyuF1RV6Ly_AVlo6Zriqb2F7vLbKUoFWzWBFC76F--0Tle9t75qfgXkVAV92ZGDw2lEL3qEbpReoOESyULakMt8ISgDmZDzKLb5wEMyvDF2CS0h9psFYpXc20MJZ6un3Vy_D33M9-NddD35-y9iTAlGhgTLXFM8m32oNL1ze_3iI_zeSf9RIfAL3OGIAfA2dfGnGWA-oC-WWJ1qV5XpdAl2MNc0XwEZQ9G-tu_FB3Zz827v7wDbDIBgcRXGQJmkQxnEcRUk66CSdoNnspkmc9JJW3B8kURK1g3Z7EHWDqJt221GzG_SCgXt-c1UUVDIX_JS7L9nSYOof_520bKYDDEdp6w6eMsLf9jrmt2yyppYXAb0EfFmnygnOlUEUGdG8r9TiIOb64JQvuKWHQuVYRkyEN4GOht0GWW6HneiwdRg6Vs24fRiBJtEkarEOdCCkwNq9o6NuRFmUd6a9ZqtNWavZy3u4--AI8gVpi_HVDVU6tlPhx6IUxybbr6ZTnHz4D1jXIAza0VG3E_jrXaOxV1cFJitTlc7hN62qEtmAZVP69tdS3gbi-hjpAqeKDAOtZT4abBSvNYdD2ScsmLpJ3aU3rzXsg0PwWWWxR6HK50N8j9WpSt_Khm6rAMaxiWxcQUXPy3ub51rN-YTbTcQj0DM4kyMlZypbN8zHuYkZGyuM8KVgXA1de0yUtFo5ilq8IROc7ozV3D3kZN-XekYeziQ2RD_nfeJ2nighPJFTpS9gBiu0iVRRC_f9Ew)

## Контрольные вопросы

### Чем отличаются totalKeysExamined и totalDocsExamined

<code>totalKeysExamined</code> — количество записей, просмотренных в индексе. <code>totalDocsExamined</code> — количество документов коллекции, которые MongoDB прочитала и проверила. Эти значения могут отличаться: индекс может просмотреть больше ключей, чем в итоге будет загружено документов.

### Зачем нужен FETCH

<code>IXSCAN</code> находит подходящие записи в индексе, но обычно в индексе нет полного содержимого документа. <code>FETCH</code> загружает сами документы коллекции по найденным ссылкам.

### Почему plan-2 просмотрел 43 ключа, а вернул 33 документа

Индексный диапазон включает 43 ключа. После проверки диапазона и фильтра MongoDB загрузила и вернула 33 подходящих документа. Остальные ключи не привели к документам результата.

### Что произойдёт с индексом при вставке новой записи

При вставке MongoDB добавит ключ нового документа в индекс. Это ускоряет чтение, но увеличивает размер индекса и стоимость операций записи.

### Почему нельзя индексировать все поля подряд

Индексы занимают место и замедляют вставки, обновления и удаления. Индекс должен соответствовать реальным запросам и правильному порядку полей, а не содержать все поля коллекции.

## Файлы

В папке находятся шесть JSON-планов, требуемых практикумом. Файл <code>plan.py</code> не добавлялся: планы снимались напрямую через <code>mongosh</code>, а не способом со скриптом.

