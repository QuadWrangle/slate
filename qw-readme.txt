You need ruby at least 2.3.1 (i recommend  using rvm for this)

To edit the documentation:

Open up source/index.html.md

Make changes

to publish:
bundle exec middleman build --clean

You will need bundle gem, and the middleman gem as well

Then upload the build/index.html to: apidocs.qwd.io (SCP)
then move it into place on that server: /var/qw20/apidocs/build


