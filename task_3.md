# Task 3
Here is pseudocode for models.py.
he database scheme is written according to the rules of normalization and i18n

```python
class Language(models.Model):
    name = models.CharField(max_length=50)
    code = models.CharField(max_length=10, unique=True)

    def __str__(self):
        return self.name

class TranslatableModel(models.Model):
    # Base model for translatable entities
    class Meta:
        abstract = True

    translations = models.ManyToManyField('Translation', related_name='%(class)s_translations')

class Translation(models.Model):
    # Model to store translations for each translatable entity
    language = models.ForeignKey(Language, on_delete=models.SET_NULL, null=True, blank=True)
    text = models.TextField()

class Skill(TranslatableModel):
    # Some fields
    pass

class Profession(TranslatableModel):
    # Some fields
    pass

class Industry(models.Model):
    # Some fields
    pass

class Course(models.Model):
    # Model for courses
    # Some fields
    title = models.CharField(max_length=255)
    description = models.TextField()
    requirements = models.TextField()
    skills = models.ManyToManyField(Skill)
    professions = models.ManyToManyField(Profession)
    industries = models.ManyToManyField(Industry)
    translations = models.ManyToManyField(Translation, related_name='course_translations')

    def __str__(self):
        return self.title

class TranslationQuality(models.TextChoices):
    GOOD = 'Good', 'Good'
    POOR = 'Poor', 'Poor'

class TranslationLog(models.Model):
    translation = models.ForeignKey(Translation, on_delete=models.CASCADE)
    original_text = models.TextField()
    quality = models.CharField(max_length=10, choices=TranslationQuality.choices)

    def __str__(self):
        return f"{self.translation.language} - {self.translation.text}"

```

