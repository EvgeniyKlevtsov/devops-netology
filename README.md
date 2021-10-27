## Описание terraform/.gitignore

    файл .gitignore будет игнорировать

 Все папки .terraform и их содержимое
 Все файлы с расширением .tfstate и/или которые содержат в своем имени подстроку .tfstate.
 Файлы журнала сбоев crash.log
 Файлы содержащие важные данные .tfvars
 Файлы локального переопределения ресурсов override.tf и override.tf.json и название которых заканчивается на _override.tf и _override.tf.json
 Конфигурационные файлы CLI .terraformrc и terraform.rc
