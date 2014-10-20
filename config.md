try:
    # Python 2.7
    from collections import OrderedDict
except:
    # Python 2.6
    from gluon.contrib.simplejson.ordered_dict import OrderedDict

from datetime import timedelta

from gluon import current, Field, URL
from gluon.html import *
from gluon.storage import Storage
from gluon.validators import IS_EMPTY_OR, IS_NOT_EMPTY

from s3.s3fields import S3Represent
from s3.s3query import FS
from s3.s3utils import S3DateTime, s3_auth_user_represent_name, s3_avatar_represent, s3_unicode
from s3.s3validators import IS_INT_AMOUNT, IS_LOCATION_SELECTOR2, IS_ONE_OF
from s3.s3widgets import S3LocationSelectorWidget2
from s3.s3forms import S3SQLCustomForm, S3SQLInlineComponent, S3SQLInlineComponentCheckbox

T = current.T
s3 = current.response.s3
settings = current.deployment_settings

datetime_represent = lambda dt: S3DateTime.datetime_represent(dt, utc=True)

"""
    Template settings for NDEM Portal
"""

# -----------------------------------------------------------------------------
# Pre-Populate
settings.base.prepopulate = ["India","demo/users"]

# =============================================================================
# System Settings
# -----------------------------------------------------------------------------
# Authorization Settings
settings.auth.registration_requires_approval = True
settings.auth.registration_requires_verification = False
settings.auth.registration_requests_organisation = True
#settings.auth.registration_organisation_required = True
settings.auth.registration_requests_site = False

# Approval emails get sent to all admins
settings.mail.approver = "ADMIN"

settings.auth.registration_link_user_to = {"staff": T("Staff")}
settings.auth.registration_link_user_to_default = ["staff"]
settings.auth.registration_roles = {"organisation_id": ["USER"],
                                    }

# Terms of Service to be able to Register on the system
# uses <template>/views/tos.html
settings.auth.terms_of_service = True

settings.auth.show_utc_offset = False

settings.auth.show_link = False

# -----------------------------------------------------------------------------
# Security Policy
#settings.security.policy = 6 # Realms
settings.security.map = True

# Owner Entity
settings.auth.person_realm_human_resource_site_then_org = False

def drmp_realm_entity(table, row):
    """
        Assign a Realm Entity to records
    """

    tablename = table._tablename

    if tablename == "cms_post":
        # Give the Post the Realm of the author's Organisation
        db = current.db
        utable = db.auth_user
        otable = current.s3db.org_organisation
        if "created_by" in row:
            query = (utable.id == row.created_by) & \
                    (otable.id == utable.organisation_id)
        else:
            query = (table.id == row.id) & \
                    (utable.id == table.created_by) & \
                    (otable.id == utable.organisation_id)
        org = db(query).select(otable.pe_id,
                               limitby=(0, 1)).first()
        if org:
            return org.pe_id

    # Follow normal rules
    return 0

settings.auth.realm_entity = drmp_realm_entity

# -----------------------------------------------------------------------------
# Theme (folder to use for views/layout.html)
settings.base.theme = "India"
settings.ui.formstyle_row = "bootstrap"
settings.ui.formstyle = "bootstrap"
#settings.gis.map_height = 600
#settings.gis.map_width = 854

# -----------------------------------------------------------------------------
# L10n (Localization) settings
settings.L10n.languages = OrderedDict([
    ("en-gb", "English"),
])
# Default Language
settings.L10n.default_language = "en-gb"
# Default timezone for users
settings.L10n.utc_offset = "UTC +0530"
# Unsortable 'pretty' date format
settings.L10n.date_format = "%d %b %Y"
# Number formats (defaults to ISO 31-0)
# Decimal separator for numbers (defaults to ,)
settings.L10n.decimal_separator = "."
# Thousands separator for numbers (defaults to space)
settings.L10n.thousands_separator = ","

# Uncomment this to Translate CMS Series Names
# - we want this on when running s3translate but off in normal usage as we use the English names to lookup icons in render_posts
#settings.L10n.translate_cms_series = True
# Uncomment this to Translate Location Names
#settings.L10n.translate_gis_location = True

# Restrict the Location Selector to just certain countries
settings.gis.countries = ["IN"]

# Until we add support to LocationSelector2 to set dropdowns from LatLons
#settings.gis.check_within_parent_boundaries = False
# Uncomment to hide Layer Properties tool
#settings.gis.layer_properties = False
# Uncomment to display the Map Legend as a floating DIV
settings.gis.legend = "float"
# GeoNames username
settings.gis.geonames_username = "eden_india"

# -----------------------------------------------------------------------------
# Finance settings
settings.fin.currencies = {
    "INR" : T("Indian Rupees"),
    "CHF" : T("Swiss Francs"),
    "EUR" : T("Euros"),
    "GBP" : T("Great British Pounds"),
    "USD" : T("United States Dollars"),
}

# -----------------------------------------------------------------------------
# Enable this for a UN-style deployment
#settings.ui.cluster = True
# Enable this to use the label 'Camp' instead of 'Shelter'
#settings.ui.camp = True

# -----------------------------------------------------------------------------
# Uncomment to restrict the export formats available
#settings.ui.export_formats = ["xls"]

settings.ui.update_label = "Edit"

# -----------------------------------------------------------------------------
# Summary Pages
settings.ui.summary = [#{"common": True,
                       # "name": "cms",
                       # "widgets": [{"method": "cms"}]
                       # },
                       {"name": "table",
                        "label": "Table",
                        "widgets": [{"method": "datatable"}]
                        },
                       {"name": "map",
                        "label": "Map",
                        "widgets": [{"method": "map", "ajax_init": True}],
                        },
                       {"name": "charts",
                        "label": "Charts",
                        "widgets": [{"method": "report", "ajax_init": True}]
                        },
                       ]

settings.search.filter_manager = False

# =============================================================================
# Module Settings

# -----------------------------------------------------------------------------
# Human Resource Management
settings.hrm.staff_label = "Contacts"
# Uncomment to allow Staff & Volunteers to be registered without an email address
settings.hrm.email_required = False
# Uncomment to show the Organisation name in HR represents
settings.hrm.show_organisation = True
# Uncomment to disable Staff experience
settings.hrm.staff_experience = False
# Uncomment to disable the use of HR Credentials
settings.hrm.use_credentials = False
# Uncomment to disable the use of HR Skills
settings.hrm.use_skills = False
# Uncomment to disable the use of HR Teams
settings.hrm.teams = False

# Uncomment to hide fields in S3AddPersonWidget[2]
settings.pr.request_dob = False
settings.pr.request_gender = False

# -----------------------------------------------------------------------------
# Org
settings.org.site_label = "Office/Shelter/Hospital"

# -----------------------------------------------------------------------------
# Project
# Uncomment this to use multiple Organisations per project
settings.project.multiple_organisations = True

# Links to Filtered Components for Donors & Partners
#settings.project.organisation_roles = {
#    1: T("Host National Society"),
#    2: T("Partner"),
#    3: T("Donor"),
#    #4: T("Customer"), # T("Beneficiary")?
#    #5: T("Supplier"),
#    9: T("Partner National Society"),
#}

# -----------------------------------------------------------------------------
# Notifications
# Template for the subject line in update notifications
#settings.msg.notify_subject = "$S %s" % T("Notification")
settings.msg.notify_subject = "$S Notification"

# -----------------------------------------------------------------------------
def currency_represent(v):
    """
        Custom Representation of Currencies
    """
    if v == "INR":
        return "RS"
    elif v == "USD":
        return "$"
    elif v == "EUR":
        return "€"
    elif v == "GBP":
        return "£"
    else:
        # e.g. CHF
        return v
def render_locations(list_id, item_id, resource, rfields, record):
    """
        Custom dataList item renderer for Locations on the Selection Page

        @param list_id: the HTML ID of the list
        @param item_id: the HTML ID of the item
        @param resource: the S3Resource to render
        @param rfields: the S3ResourceFields to render
        @param record: the record as dict
    """

    record_id = record["gis_location.id"]
    item_class = "thumbnail"

    raw = record._row
    name = raw["gis_location.name"]
    level = raw["gis_location.level"]
    L1 = raw["gis_location.L1"]
    L2 = raw["gis_location.L2"]
    L3 = raw["gis_location.L3"]
    location_url = URL(c="gis", f="location",
                       args=[record_id, "profile"])

    if level == "L1":
        represent = name
    if level == "L2":
        represent = "%s (%s)" % (name, L1)
    elif level == "L3":
        represent = "%s (%s, %s)" % (name, L2, L1)
    else:
        # L0 or specific
        represent = name

    # Users don't edit locations
    # permit = current.auth.s3_has_permission
    # table = current.db.gis_location
    # if permit("update", table, record_id=record_id):
        # edit_btn = A(I(" ", _class="icon icon-edit"),
                     # _href=URL(c="gis", f="location",
                               # args=[record_id, "update.popup"],
                               # vars={"refresh": list_id,
                                     # "record": record_id}),
                     # _class="s3_modal",
                     # _title=current.response.s3.crud_strings.gis_location.title_update,
                     # )
    # else:
        # edit_btn = ""
    # if permit("delete", table, record_id=record_id):
        # delete_btn = A(I(" ", _class="icon icon-remove-sign"),
                       # _class="dl-item-delete",
                      # )
    # else:
        # delete_btn = ""
    # edit_bar = DIV(edit_btn,
                   # delete_btn,
                   # _class="edit-bar fright",
                   # )

    # Tallies
    # NB We assume that all records are readable here
    # Search all sub-locations
    locations = current.gis.get_children(record_id)
    locations = [l.id for l in locations]
    locations.append(record_id)
    db = current.db
    s3db = current.s3db
    ltable = s3db.project_location
    table = db.project_project
    query = (table.deleted == False) & \
            (ltable.deleted == False) & \
            (ltable.project_id == table.id) & \
            (ltable.location_id.belongs(locations))
    rows = db(query).select(table.id, distinct=True)
    tally_projects = len(rows)
    tally_incidents = 0
    tally_activities = 0
    tally_reports = 0
    table = s3db.cms_post
    stable = db.cms_series
    types = ["Alert", "Activity", "Report"]
    query = (table.deleted == False) & \
            (table.location_id.belongs(locations)) & \
            (stable.id == table.series_id) & \
            (stable.name.belongs(types))
    rows = db(query).select(stable.name)
    for row in rows:
        series = row.name
        if series == "Alert":
            tally_incidents += 1
        elif series == "Activity":
            tally_activities += 1
        elif series == "Report":
            tally_reports += 1

    # Build the icon, if it doesn't already exist
    filename = "%s.svg" % record_id
    import os
    filepath = os.path.join(current.request.folder, "static", "cache", "svg", filename)
    if not os.path.exists(filepath):
        gtable = db.gis_location
        loc = db(gtable.id == record_id).select(gtable.wkt,
                                                limitby=(0, 1)
                                                ).first()
        if loc:
            from s3.s3codecs.svg import S3SVG
            S3SVG.write_file(filename, loc.wkt)

    # Render the item
    item = DIV(DIV(A(IMG(_class="media-object",
                         _src=URL(c="static",
                                  f="cache",
                                  args=["svg", filename],
                                  )
                         ),
                     _class="pull-left",
                     _href=location_url,
                     ),
                   DIV(SPAN(A(represent,
                              _href=location_url,
                              _class="media-heading"
                              ),
                            ),
                       #edit_bar,
                       _class="card-header-select",
                       ),
                   DIV(P(T("Alerts"),
                         SPAN(tally_incidents,
                              _class="badge",
                              ),
                         #T("Reports"),
                         #SPAN(tally_reports,
                              #_class="badge",
                              #),
                         T("Projects"),
                         SPAN(tally_projects,
                              _class="badge",
                              ),
                         #T("Activities"),
                         #SPAN(tally_activities,
                              #_class="badge",
                              #),
                         _class="tally",
                         ),
                       _class="media-body",
                       ),
                   _class="media",
                   ),
               _class=item_class,
               _id=item_id,
               )

    return item

# -----------------------------------------------------------------------------
def render_locations_profile(list_id, item_id, resource, rfields, record):
    """
        Custom dataList item renderer for Locations on the Profile Page
        - UNUSED

        @param list_id: the HTML ID of the list
        @param item_id: the HTML ID of the item
        @param resource: the S3Resource to render
        @param rfields: the S3ResourceFields to render
        @param record: the record as dict
    """

    record_id = record["gis_location.id"]
    item_class = "thumbnail"

    raw = record._row
    name = record["gis_location.name"]
    location_url = URL(c="gis", f="location",
                       args=[record_id, "profile"])

    # Placeholder to maintain style
    #logo = DIV(IMG(_class="media-object"),
    #               _class="pull-left")

    # We don't Edit Locations
    # Edit Bar
    # permit = current.auth.s3_has_permission
    # table = current.db.gis_location
    # if permit("update", table, record_id=record_id):
        # vars = {"refresh": list_id,
                # "record": record_id,
                # }
        # f = current.request.function
        # if f == "organisation" and organisation_id:
            # vars["(organisation)"] = organisation_id
        # edit_btn = A(I(" ", _class="icon icon-edit"),
                     # _href=URL(c="gis", f="location",
                               # args=[record_id, "update.popup"],
                               # vars=vars),
                     # _class="s3_modal",
                     # _title=current.response.s3.crud_strings.gis_location.title_update,
                     # )
    # else:
        # edit_btn = ""
    # if permit("delete", table, record_id=record_id):
        # delete_btn = A(I(" ", _class="icon icon-remove-sign"),
                       # _class="dl-item-delete",
                       # )
    # else:
        # delete_btn = ""
    # edit_bar = DIV(edit_btn,
                   # delete_btn,
                   # _class="edit-bar fright",
                   # )

    # Render the item
    item = DIV(DIV(DIV(#SPAN(A(name,
                       #       _href=location_url,
                       #       ),
                       #     _class="location-title"),
                       #" ",
                       #edit_bar,
                       P(A(name,
                           _href=location_url,
                           ),
                         _class="card_comments"),
                       _class="span5"), # card-details
                   _class="row",
                   ),
               )

    return item
def render_projects(list_id, item_id, resource, rfields, record):
    """
        Custom dataList item renderer for Projects on the Profile pages

        @param list_id: the HTML ID of the list
        @param item_id: the HTML ID of the item
        @param resource: the S3Resource to render
        @param rfields: the S3ResourceFields to render
        @param record: the record as dict
    """

    record_id = record["project_project.id"]
    item_class = "thumbnail"

    raw = record._row
    name = record["project_project.name"]
    author = record["project_project.modified_by"]
    author_id = raw["project_project.modified_by"]
    contact = record["project_project.human_resource_id"]
    date = record["project_project.modified_on"]
    organisation = record["project_project.organisation_id"]
    organisation_id = raw["project_project.organisation_id"]
    location = record["project_location.location_id"]
    location_ids = raw["project_location.location_id"]
    if isinstance(location_ids, list):
        locations = location.split(",")
        locations_list = []
        length = len(location_ids)
        i = 0
        for location_id in location_ids:
            location_url = URL(c="gis", f="location",
                               args=[location_id, "profile"])
            locations_list.append(A(locations[i], _href=location_url))
            i += 1
            if i != length:
                locations_list.append(",")
    else:
        location_url = URL(c="gis", f="location",
                           args=[location_ids, "profile"])
        locations_list = [A(location, _href=location_url)]

    logo = raw["org_organisation.logo"]
    org_url = URL(c="org", f="organisation", args=[organisation_id, "profile"])
    if logo:
        avatar = A(IMG(_src=URL(c="default", f="download", args=[logo]),
                       _class="media-object",
                       ),
                   _href=org_url,
                   _class="pull-left",
                   )
    else:
        avatar = DIV(IMG(_class="media-object"),
                     _class="pull-left")

    db = current.db
    s3db = current.s3db
    ltable = s3db.pr_person_user
    ptable = db.pr_person
    query = (ltable.user_id == author_id) & \
            (ltable.pe_id == ptable.pe_id)
    row = db(query).select(ptable.id,
                           limitby=(0, 1)
                           ).first()
    if row:
        person_url = URL(c="hrm", f="person", args=[row.id])
    else:
        person_url = "#"
    author = A(author,
               _href=person_url,
               )

    start_date = raw["project_project.start_date"] or ""
    if start_date:
        start_date = record["project_project.start_date"]
    end_date = raw["project_project.end_date"] or ""
    if end_date:
        end_date = record["project_project.end_date"]
    budget = record["project_project.budget"]
    if budget:
        budget = "USD %s" % budget

    partner = record["project_partner_organisation.organisation_id"]
    partner_ids = raw["project_partner_organisation.organisation_id"]
    if isinstance(partner_ids, list):
        partners = partner.split(",")
        partners_list = []
        length = len(partner_ids)
        i = 0
        for partner_id in partner_ids:
            partner_url = URL(c="org", f="organisation",
                              args=[partner_id, "profile"])
            partners_list.append(A(partners[i], _href=partner_url))
            i += 1
            if i != length:
                partners_list.append(",")
    elif partner_ids:
        partner_url = URL(c="org", f="organisation",
                          args=[partner_ids, "profile"])
        partners_list = [A(partner, _href=partner_url)]
    else:
        partners_list = [current.messages["NONE"]]

    donor = record["project_donor_organisation.organisation_id"]
    donor_ids = raw["project_donor_organisation.organisation_id"]
    if isinstance(donor_ids, list):
        donors = donor.split(",")
        amounts = raw["project_donor_organisation.amount"]
        if not isinstance(amounts, list):
            amounts = [amounts for donor_id in donor_ids]
        currencies = raw["project_donor_organisation.currency"]
        if not isinstance(currencies, list):
            currencies = [currencies for donor_id in donor_ids]
        amount_represent = IS_INT_AMOUNT.represent
        donors_list = []
        length = len(donor_ids)
        i = 0
        for donor_id in donor_ids:
            if donor_id:
                donor_url = URL(c="org", f="organisation",
                                args=[donor_id, "profile"])
                donor = A(donors[i], _href=donor_url)
                amount = amounts[i]
                if amount:
                    donor = TAG[""](donor,
                                    " - ",
                                    currency_represent(currencies[i]),
                                    amount_represent(amount))
            else:
                donor = current.messages["NONE"]
            donors_list.append(donor)
            i += 1
            if i != length:
                donors_list.append(",")
    elif donor_ids:
        donor_url = URL(c="org", f="organisation",
                      args=[donor_ids, "profile"])
        donors_list = [A(donor, _href=donor_url)]
    else:
        donors_list = [current.messages["NONE"]]

    # Edit Bar
    permit = current.auth.s3_has_permission
    table = current.db.project_project
    if permit("update", table, record_id=record_id):
        vars = {"refresh": list_id,
                "record": record_id,
                }
        f = current.request.function
        if f == "organisation" and organisation_id:
            vars["(organisation)"] = organisation_id
        # "record not found" since multiples here
        #elif f == "location" and location_ids:
        #    vars["(location)"] = location_ids
        edit_btn = A(I(" ", _class="icon icon-edit"),
                     _href=URL(c="project", f="project",
                               args=[record_id, "update.popup"],
                               vars=vars),
                     _class="s3_modal",
                     _title=current.response.s3.crud_strings.project_project.title_update,
                     )
    else:
        # Read in Popup
        edit_btn = A(I(" ", _class="icon icon-search"),
                     _href=URL(c="project", f="project",
                               args=[record_id, "read.popup"]),
                     _class="s3_modal",
                     _title=current.response.s3.crud_strings.project_project.title_display,
                     )
    if permit("delete", table, record_id=record_id):
        delete_btn = A(I(" ", _class="icon icon-remove-sign"),
                       _class="dl-item-delete",
                       )
    else:
        delete_btn = ""
    edit_bar = DIV(edit_btn,
                   delete_btn,
                   _class="edit-bar fright",
                   )

    # Dropdown of available documents
    documents = raw["doc_document.file"]
    if documents:
        if not isinstance(documents, list):
            documents = [documents]
        doc_list = UL(_class="dropdown-menu",
                      _role="menu",
                      )
        retrieve = db.doc_document.file.retrieve
        for doc in documents:
            try:
                doc_name = retrieve(doc)[0]
            except IOError:
                doc_name = current.messages["NONE"]
            doc_url = URL(c="default", f="download",
                          args=[doc])
            doc_item = LI(A(I(_class="icon-file"),
                            " ",
                            doc_name,
                            _href=doc_url,
                            ),
                          _role="menuitem",
                          )
            doc_list.append(doc_item)
        docs = DIV(A(I(_class="icon-paper-clip"),
                     SPAN(_class="caret"),
                     _class="btn dropdown-toggle",
                     _href="#",
                     **{"_data-toggle": "dropdown"}
                     ),
                   doc_list,
                   _class="btn-group attachments dropdown pull-right",
                   )
    else:
        docs = ""

    # Render the item,
    body = TAG[""](P(I(_class="icon-user"),
                     " ",
                     STRONG("%s:" % T("Focal Point")),
                     " ",
                     contact,
                     _class="main_contact_ph"),
                   P(I(_class="icon-calendar"),
                     " ",
                     STRONG("%s:" % T("Start & End Date")),
                     " ",
                     T("%(start_date)s to %(end_date)s") % \
                            dict(start_date=start_date,
                            end_date = end_date),
                     _class="main_contact_ph"),
                   P(I(_class="icon-link"),
                     " ",
                     STRONG("%s:" % T("Partner")),
                     " ",
                     *partners_list,
                     _class="main_contact_ph"),
                   P(I(_class="icon-money"),
                     " ",
                     STRONG("%s:" % T("Donor")),
                     " ",
                     *donors_list,
                     _class="main_office-add")
                   )

    item = DIV(DIV(SPAN(" ", _class="card-title"),
                   SPAN(*locations_list,
                        _class="location-title"
                        ),
                   SPAN(date,
                        _class="date-title",
                        ),
                   edit_bar,
                   _class="card-header",
                   ),
               DIV(avatar,
                   DIV(DIV(body,
                           DIV(author,
                               " - ",
                               A(organisation,
                                 _href=org_url,
                                 _class="card-organisation",
                                 ),
                               _class="card-person",
                               ),
                           _class="media",
                           ),
                       _class="media-body",
                       ),
                   _class="media",
                   ),
               docs,
               _class=item_class,
               _id=item_id,
               )

    return item
def customise_gis_location_controller(**attr):
    """
        Customise gis_location controller
        - Profile Page
    """

    s3 = current.response.s3

    # Custom PreP
    standard_prep = s3.prep
    def custom_prep(r):
        if r.interactive:
            s3db = current.s3db
            table = s3db.gis_location

            s3.crud_strings["gis_location"].title_list = T("States")

            if r.method == "datalist":
                # District selection page
                # 2-column datalist, 6 rows per page
                s3.dl_pagelength = 12
                s3.dl_rowsize = 2

                # Just show In L1s
                s3.filter = (table.L0 == "India") & (table.level == "L1")
                # Default 5 triggers an AJAX call, we should load all by default
                s3.dl_pagelength = 13

                list_fields = ["name",
                               "level",
                               "L1",
                               "L2",
                               "L3",
                               ]
                s3db.configure("gis_location",
                               list_fields = list_fields,
                               list_layout = render_locations,
                               )

            elif r.method == "profile":
        
                # Customise tables used by widgets
                customise_cms_post_fields()
                customise_project_project_fields()

                # gis_location table (Sub-Locations)
                table.parent.represent = s3db.gis_LocationRepresent(sep=" | ")

                list_fields = ["name",
                               "id",
                               ]

                location = r.record
                record_id = location.id
                default = "~.(location)=%s" % record_id
                map_widget = dict(label = "Map",
                                  type = "map",
                                  context = "location",
                                  icon = "icon-map",
                                  height = 383,
                                  width = 568,
                                  bbox = {"lat_max" : location.lat_max,
                                          "lon_max" : location.lon_max,
                                          "lat_min" : location.lat_min,
                                          "lon_min" : location.lon_min
                                          },
                                  )
                #locations_widget = dict(label = "Locations",
                #                        insert = False,
                #                        #label_create = "Create Location",
                #                        type = "datalist",
                #                        tablename = "gis_location",
                #                        context = "location",
                #                        icon = "icon-globe",
                #                        # @ToDo: Show as Polygons?
                #                        show_on_map = False,
                #                        list_layout = render_locations_profile,
                #                        )
                incidents_widget = dict(label = "Alerts",
                                        label_create = "Add New Alert",
                                        type = "datalist",
                                        tablename = "cms_post",
                                        context = "location",
                                        default = default,
                                        filter = (FS("series_id$name") == "Alert") & (FS("expired") == False),
                                        icon = "icon-alert",
                                        layer = "Incidents",
                                        # provided by Catalogue Layer
                                        #marker = "incident",
                                        list_layout = render_profile_posts,
                                        )
                projects_widget = dict(label = "Projects",
                                       label_create = "Create Project",
                                       type = "datalist",
                                       tablename = "project_project",
                                       context = "location",
                                       default = default,
                                       icon = "icon-project",
                                       show_on_map = False, # No Marker yet & only show at L1-level anyway
                                       list_layout = render_projects,
                                       )
                reports_widget = dict(label = "Reports",
                                      label_create = "Create Report",
                                      type = "datalist",
                                      tablename = "cms_post",
                                      context = "location",
                                      default = default,
                                      filter = FS("series_id$name") == "Report",
                                      icon = "icon-report",
                                      layer = "Reports",
                                      # provided by Catalogue Layer
                                      #marker = "report",
                                      list_layout = render_profile_posts,
                                      )
                # @ToDo: Renderer
                #distributions_widget = dict(label = "Distributions",
                #                            label_create = "Create Distribution",
                #                            type = "datalist",
                #                            tablename = "supply_distribution",
                #                            context = "location",
                #                            default = default,
                #                            icon = "icon-resource",
                #                            list_layout = render_distributions,
                #                            )
                # Build the icon, if it doesn't already exist
                filename = "%s.svg" % record_id
                import os
                filepath = os.path.join(current.request.folder, "static", "cache", "svg", filename)
                if not os.path.exists(filepath):
                    gtable = s3db.gis_location
                    db = current.db
                    loc = db(gtable.id == record_id).select(gtable.wkt,
                                                            limitby=(0, 1)
                                                            ).first()
                    if loc:
                        from s3.s3codecs.svg import S3SVG
                        S3SVG.write_file(filename, loc.wkt)

                name = location.name
                s3db.configure("gis_location",
                               list_fields = list_fields,
                               profile_title = "%s : %s" % (s3.crud_strings["gis_location"].title_list, 
                                                            name),
                               profile_header = DIV(A(IMG(_class="media-object",
                                                          _src=URL(c="static",
                                                                   f="cache",
                                                                   args=["svg", filename],
                                                                   ),
                                                          ),
                                                      _class="pull-left",
                                                      #_href=location_url,
                                                      ),
                                                    H2(name),
                                                    _class="profile-header",
                                                    ),
                               profile_widgets = [#locations_widget,
                                                  incidents_widget,
                                                  #map_widget,
                                                  projects_widget,
                                                  #reports_widget,
                                                  #activities_widget,
                                                  #distributions_widget,
                                                  ],
                               )

        # Call standard prep
        if callable(standard_prep):
            result = standard_prep(r)
            if not result:
                return False

        return True
    s3.prep = custom_prep

    return attr

settings.customise_gis_location_controller = customise_gis_location_controller
def customise_project_project_fields():
    """
        Customise project_project fields for Profile widgets and 'more' popups
    """

    format = "%d/%m/%y"
    date_represent = lambda d: S3DateTime.date_represent(d, format=format)

    s3db = current.s3db

    s3db.project_location.location_id.represent = s3db.gis_LocationRepresent(sep=" | ")
    table = s3db.project_project
    table.start_date.represent = date_represent
    table.end_date.represent = date_represent
    table.modified_by.represent = s3_auth_user_represent_name
    table.modified_on.represent = datetime_represent

    list_fields = [#"name",
                   "organisation_id",
                   "location.location_id",
                   "organisation_id$logo",
                   "start_date",
                   "end_date",
                   #"human_resource_id",
                   "budget",
                   "partner.organisation_id",
                   "donor.organisation_id",
                   "donor.amount",
                   "donor.currency",
                   "modified_by",
                   "modified_on",
                   "document.file",
                   ]

    s3db.configure("project_project",
                   list_fields = list_fields,
                   )

# ----------------------------------------------------------------------

# -----------------------------------------------------------------------------

def customise_project_project_controller(**attr):

    s3 = current.response.s3
    s3db = current.s3db
    table = s3db.project_project
    tablename = "project_project"
    # Remove rheader
    attr["rheader"] = None

    # Custom PreP
    standard_prep = s3.prep
    
    def custom_prep(r):
        # Call standard prep
        if callable(standard_prep):
            result = standard_prep(r)
            if not result:
                return False

        #s3db = current.s3db
        #table = s3db.project_project
		
        if r.method == "datalist":
            customise_project_project_fields()
            s3db.configure(tablename,
                           # Don't include a Create form in 'More' popups
                           listadd = False,
                           list_layout = render_projects,
                           )
		
        elif r.interactive or r.representation == "aadata":
		#if r.interactive or r.representation == "aadata":
			#customise_project_project_fields(r.method)
            # Filter from a Profile page?
            # If so, then default the fields we know
            get_vars = current.request.get_vars
            organisation_id = get_vars.get("~.(organisation)", None)
            if not organisation_id:
                user = current.auth.user
                if user:
                    organisation_id = user.organisation_id

            # Configure fields 
            table.objectives.readable = table.objectives.writable = True
            table.human_resource_id.label = T("Focal Person")
            s3db.hrm_human_resource.organisation_id.default = organisation_id
            table.budget.label = "%s (USD)" % T("Budget")
            # Better in column label & otherwise this construction loses thousands separators
            #table.budget.represent = lambda value: "%d USD" % value

            crud_form_fields = [
                    "name",
                    S3SQLInlineComponentCheckbox(
                        "theme",
                        label = T("Themes"),
                        field = "theme_id",
                        option_help = "comments",
                        cols = 3,
                    ),
                    S3SQLInlineComponent(
                        "location",
                        label = T("State"),
                        fields = [("", "location_id")],
                        orderby = "location_id$name",
                        render_list = True
                    ),
                    "description",
                    "human_resource_id",
                    "start_date",
                    "end_date",
                    # Partner Orgs
                    S3SQLInlineComponent(
                        "organisation",
                        name = "partner",
                        label = T("Partner Organizations"),
                        fields = [("", "organisation_id")],
                        filterby = dict(field = "role",
                                        options = "2"
                                        )
                    ),
                    # Donors
                    S3SQLInlineComponent(
                        "organisation",
                        name = "donor",
                        label = T("Donor(s)"),
                        fields = ["organisation_id", "amount", "currency"],
                        filterby = dict(field = "role",
                                        options = "3"
                                        )
                    ),
                    "budget",
                    #"objectives",
                    # Files
                    S3SQLInlineComponent(
                        "document",
                        name = "file",
                        label = T("Files"),
                        fields = [("", "file"),
                                  #"comments"
                                  ],
                    ),
                    "comments",
                    ]
            if organisation_id:
                org_field = table.organisation_id
                org_field.default = organisation_id
                org_field.readable = org_field.writable = False
            else:
                location_field = s3db.project_location.location_id
                location_id = get_vars.get("~.(location)", None)
                if location_id:
                    # Default to this Location, but allow selection of others
                    location_field.default = location_id
                location_field.label = ""
                location_field.represent = S3Represent(lookup="gis_location")
                # Project Locations must be districts
                location_field.requires = IS_ONE_OF(current.db, "gis_location.id",
                                                    S3Represent(lookup="gis_location"),
                                                    sort = True,
                                                    filterby = "level",
                                                    filter_opts = ("L1",)
                                                    )
                # Don't add new Locations here
                location_field.comment = None
                # Simple dropdown
                location_field.widget = None
                crud_form_fields.insert(1, "organisation_id")

            crud_form = S3SQLCustomForm(*crud_form_fields)

            list_fields = ["name",
                           "organisation_id",
                           "human_resource_id",
                           (T("States"), "location.location_id"),
                           "start_date",
                           "end_date",
                           #"budget",
                           ]

            # Return to List view after create/update/delete (unless done via Modal)
            url_next = URL(c="project", f="project")

            from s3.s3filter import S3LocationFilter, S3TextFilter, S3OptionsFilter
            filter_widgets = [
                S3TextFilter(["name",
                              "description",
                              "location.location_id",
                              #"theme.name",
                              #"objectives",
                              "comments"
                              ],
                             label = T("Search Projects"),
                             ),
                S3OptionsFilter("organisation_id",
                                label = T("Lead Organization"),
                                ),
                S3LocationFilter("location.location_id",
                                 levels=["L1", "L2", "L3"],
								 label=T("Location")
                                 ),
                S3OptionsFilter("partner.organisation_id",
                                label = T("Partners"),
                                ),
                S3OptionsFilter("donor.organisation_id",
                                label = T("Donors"),
                                )
                ]
        
            s3db.configure(tablename,
                           create_next = url_next,
                           delete_next = url_next,
                           update_next = url_next,
                           crud_form = crud_form,
                           list_fields = list_fields,
						   listadd = False if r.method=="datalist" else True,
						   list_layout = render_projects,
                           filter_widgets = filter_widgets,
                           )
        
            # This is awful in Popups & inconsistent in dataTable view (People/Documents don't have this & it breaks the styling of the main Save button)
            #s3.cancel = URL(c="project", f="project")

        return True
    s3.prep = custom_prep
	
    # Custom postp
    standard_postp = s3.postp
    def custom_postp(r, output):
		#s3db = current.s3db
        #table = s3db.project_project
        if r.interactive:
            actions = [dict(label=str(T("Open")),
                            _class="action-btn",
                            url=URL(c="project", f="project",
                                    args=["[id]", "read"]))
                       ]
            # All users just get "Open"
            #db = current.db
            #auth = current.auth
            #has_permission = auth.s3_has_permission
            #ownership_required = auth.permission.ownership_required
            #s3_accessible_query = auth.s3_accessible_query
            #if has_permission("update", table):
            #    action = dict(label=str(T("Edit")),
            #                  _class="action-btn",
            #                  url=URL(c="project", f="project",
            #                          args=["[id]", "update"]),
            #                  )
            #    if ownership_required("update", table):
            #        # Check which records can be updated
            #        query = s3_accessible_query("update", table)
            #        rows = db(query).select(table._id)
            #        restrict = []
            #        rappend = restrict.append
            #        for row in rows:
            #            row_id = row.get("id", None)
            #            if row_id:
            #                rappend(str(row_id))
            #        action["restrict"] = restrict
            #    actions.append(action)
            #if has_permission("delete", table):
            #    action = dict(label=str(T("Delete")),
            #                  _class="action-btn",
            #                  url=URL(c="project", f="project",
            #                          args=["[id]", "delete"]),
            #                  )
            #    if ownership_required("delete", table):
            #        # Check which records can be deleted
            #        query = s3_accessible_query("delete", table)
            #        rows = db(query).select(table._id)
            #        restrict = []
            #        rappend = restrict.append
            #        for row in rows:
            #            row_id = row.get("id", None)
            #            if row_id:
            #                rappend(str(row_id))
            #        action["restrict"] = restrict
            #    actions.append(action)
            s3.actions = actions
            if isinstance(output, dict):
                if "form" in output:
                    output["form"].add_class("project_project")
                elif "item" in output and hasattr(output["item"], "add_class"):
                    output["item"].add_class("project_project")

        # Call standard postp
        if callable(standard_postp):
            output = standard_postp(r, output)

        return output
    s3.postp = custom_postp

    return attr

settings.customise_project_project_controller = customise_project_project_controller

# -----------------------------------------------------------------------------
# Filter forms - style for Summary pages
def filter_formstyle(row_id, label, widget, comment, hidden=False):
    return DIV(label, widget, comment, 
               _id=row_id,
               _class="horiz_filter_form")

# =============================================================================
# Template Modules
# Comment/uncomment modules here to disable/enable them
settings.modules = OrderedDict([
    # Core modules which shouldn't be disabled
    ("default", Storage(
        name_nice = "Home",
        restricted = False, # Use ACLs to control access to this module
        access = None,      # All Users (inc Anonymous) can see this module in the default menu & access the controller
        module_type = None  # This item is not shown in the menu
    )),
    ("admin", Storage(
        name_nice = "Administration",
        #description = "Site Administration",
        restricted = True,
        access = "|1|",     # Only Administrators can see this module in the default menu & access the controller
        module_type = None  # This item is handled separately for the menu
    )),
    ("appadmin", Storage(
        name_nice = "Administration",
        #description = "Site Administration",
        restricted = True,
        module_type = None  # No Menu
    )),
    ("errors", Storage(
        name_nice = "Ticket Viewer",
        #description = "Needed for Breadcrumbs",
        restricted = False,
        module_type = None  # No Menu
    )),
    ("sync", Storage(
        name_nice = "Synchronization",
        #description = "Synchronization",
        restricted = True,
        access = "|1|",     # Only Administrators can see this module in the default menu & access the controller
        module_type = None  # This item is handled separately for the menu
    )),
    ("translate", Storage(
        name_nice = "Translation Functionality",
        #description = "Selective translation of strings based on module.",
        module_type = None,
    )),
    ("gis", Storage(
        name_nice = "Map",
        #description = "Situation Awareness & Geospatial Analysis",
        restricted = True,
        module_type = 1,     # 1st item in the menu
    )),
    ("pr", Storage(
        name_nice = "Persons",
        #description = "Central point to record details on People",
        restricted = True,
        access = "|1|",     # Only Administrators can see this module in the default menu (access to controller is possible to all still)
        module_type = None
    )),
    ("org", Storage(
        name_nice = "Organizations",
        #description = 'Lists "who is doing what & where". Allows relief agencies to coordinate their activities',
        restricted = True,
        module_type = None
    )),
    # All modules below here should be possible to disable safely
    ("hrm", Storage(
        name_nice = "Contacts",
        #description = "Human Resources Management",
        restricted = True,
        module_type = None,
    )),
    ("cms", Storage(
            name_nice = "Content Management",
            restricted = True,
            module_type = None,
        )),
    ("doc", Storage(
        name_nice = "Documents",
        #description = "A library of digital resources, such as photos, documents and reports",
        restricted = True,
        module_type = None,
    )),
    ("msg", Storage(
        name_nice = "Messaging",
        #description = "Sends & Receives Alerts via Email & SMS",
        restricted = True,
        # The user-visible functionality of this module isn't normally required. Rather it's main purpose is to be accessed from other modules.
        module_type = None,
    )),
    ("event", Storage(
        name_nice = "Disasters",
        #description = "Events",
        restricted = True,
        module_type = None
    )),
    ("project", Storage(
        name_nice = "Projects",
        restricted = True,
        module_type = None
    )),
    ("stats", Storage(
        name_nice = "Statistics",
        restricted = True,
        module_type = None
    )),
    ("vulnerability", Storage(
        name_nice = "Vulnerability",
        restricted = True,
        module_type = None
    )),
    ("transport", Storage(
        name_nice = "Transport",
        restricted = True,
        module_type = None
    )),
    ("hms", Storage(
        name_nice = "Hospitals",
        restricted = True,
        module_type = None
    )),
    ("cr", Storage(
        name_nice = "Shelters",
        restricted = True,
        module_type = None
    )),
    ("supply", Storage(
        name_nice = "Supply Chain Management",
        restricted = True,
        module_type = None
    )),
    ("inv", Storage(
        name_nice = "Resource Inventory",
        restricted = True,
        module_type = None
    )),
    ("irs", Storage(
        name_nice = "Incident Report",
        restricted = True,
        module_type = None
    )),
    ("vehicle", Storage(
        name_nice = "Vehicles",
        restricted = True,
        module_type = None
    )),
    ("asset", Storage(
        name_nice = "Assets",
        restricted = True,
        module_type = None
    )),
])
