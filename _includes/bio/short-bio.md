{% assign now = 'now' | date: '%s' %}
{% assign careerStart = '01/09/2007' | date: '%s'%}
{% assign yearsOfExperience = now | minus: careerStart | date: '%Y' | minus: 1970 %}

Hanno Embregts is a Java Developer, Speaker and Teacher at Info Support (the Netherlands). He has over {{ yearsOfExperience }} years of experience with both front- and back-end development, with a special interest in automating the software development process to the fullest.